# 041 — Lint Operation

## Status

Design draft. Not yet implemented.

## What lint is

Karpathy's original framing is direct: *"Periodically, ask the LLM to health-check the wiki. Look for: contradictions between pages, stale claims that newer sources have superseded, orphan pages with no inbound links, important concepts mentioned but lacking their own page, missing cross-references, data gaps."*

The existing `wiki-health` script handles **structural** health — broken links, missing provenance frontmatter, orphan index entries, stale log, inbox backlog. These are mechanical checks a script can run reliably.

Lint is the complementary **semantic** health pass. It catches things that look structurally fine but are wrong or incomplete in meaning: a map cluster that says "pending ingest" when ingest is done; a concept page that contradicts a newer source; a source ingested but never cross-linked anywhere; navigational pages that have drifted from wiki reality.

The distinction matters because:
- `wiki-health` runs every maintenance cycle — fast, deterministic, cheap
- lint runs less frequently and requires an LLM — slower, context-heavy, expensive

## What triggered this design note

During an IONOS wiki session (2026-04-27), the overview map at `wiki/maps/ionos-knowledgebase-overview.md` had clusters 2–7 marked as `<!-- populated during first ingest pass -->` and "pending first standing-source ingest pass". The ingest had long since completed — 69 source pages and 21 concept pages existed — but the map was never updated. `wiki-health` passed cleanly because no links were broken and no structural rules were violated. The staleness was purely semantic.

This is the canonical lint failure mode: **the wiki is internally consistent but externally wrong**.

---

## Lint check categories

### Category 1 — Navigational staleness (script-detectable)

These can be caught by extending `wiki-health` with pattern matching. No LLM required.

| Check | Signal | Detection |
|-------|--------|-----------|
| Map stubs post-ingest | Map page contains `<!-- populated`, `pending.*ingest`, `coming soon` | `grep` on all map pages |
| Empty cluster | Map section has no wikilinks | Parse map headings; flag sections with zero `[[...]]` links |
| Dead section header | Section exists in map but no corresponding source/concept pages found in wiki | Cross-reference map section keywords against `wiki/sources/` and `wiki/concepts/` filenames |

**Recommendation:** add these as new `[WARN]` checks in `wiki-health`. They are cheap, deterministic, and catch exactly the IONOS map failure.

### Category 2 — Coverage drift (script-detectable)

Sources or concepts exist that are not referenced from any map or index page.

| Check | Signal | Detection |
|-------|--------|-----------|
| Unmapped source | Source page exists in `wiki/sources/` but slug not found in any map file | `grep -rL` across `wiki/maps/` for each source slug |
| Unmapped concept | Concept page exists but not referenced from `wiki/concepts.md` or any map | Same approach |
| Index gap | Page in `wiki/sources/` or `wiki/concepts/` not listed in `wiki/sources.md` / `wiki/concepts.md` | Already partially in `wiki-health`; extend to maps |

**Recommendation:** add to `wiki-health` as `[WARN]` level. Catches accumulation drift as the wiki grows without requiring any LLM pass.

### Category 3 — Semantic staleness (LLM-required)

These require reading and reasoning about content. Not scriptable.

| Check | What it looks for |
|-------|-------------------|
| Stale claim | A concept or source page makes a claim that a newer source has superseded or contradicted |
| Orphan mention | A named entity or concept is referenced in multiple sources but has no concept page |
| Weak synthesis | A concept page describes only one source; synthesis across available sources would be richer |
| Missing cross-link | Two concept pages are closely related but don't link to each other |
| Map narrative drift | A map section's prose description no longer matches the concepts and sources it links to |
| Outdated summary | A source page summary is accurate to the original source but a newer source has made part of it obsolete |

These cannot be expressed as scripts. They require an LLM to read pages and compare content.

### Category 4 — Structural conventions (partially scriptable)

| Check | Detection |
|-------|-----------|
| Provenance missing or malformed | Already in `wiki-health` |
| Log entry format inconsistent | `grep` on `wiki/log.md` for non-conforming entries |
| Concept slug vs filename mismatch | Check that `[[wikilink]]` slugs resolve to actual filenames |
| Template fields missing | Compare source/concept page frontmatter against expected schema |

---

## Recommended implementation plan

### Phase 1 — Extend `wiki-health` (do now)

Add the script-detectable Category 1 and Category 2 checks to the `wiki-health` script. These are low-cost, high-value, and catch the failure mode already observed in production.

New checks to add:
- `[stub-map]` — map page contains known stub patterns (`<!-- populated`, `pending.*ingest`)
- `[empty-cluster]` — map section with heading but zero wikilinks
- `[unmapped-source]` — source page slug not referenced in any map
- `[unmapped-concept]` — concept page not referenced in `wiki/concepts.md` or any map

These would surface as `WARN` in normal output, `ERROR` for the most critical (e.g. a stub map pattern should probably be `WARN`).

### Phase 2 — LLM lint pass as a maintenance operation (design later)

A periodic LLM-driven lint pass covering Category 3. This is not a daily cron operation — it is a deliberate, heavier review. Candidate triggers:

1. **Manual** — the human asks for a lint pass explicitly (`/wiki-lint` or equivalent)
2. **Volume-triggered** — the maintenance cron flags a lint pass when source count or concept count has grown by more than N since the last lint log entry
3. **Quarterly scheduled** — a separate cron job at low frequency (monthly or quarterly) that runs a lightweight LLM lint prompt and reports findings

The lint prompt would:
1. Read `wiki/index.md` and `wiki/maps/` to understand current structure
2. Sample concept pages (e.g. newest 10, randomly selected 10)
3. Check sampled pages against the Category 3 checks above
4. Report findings to `wiki/log.md` and optionally to a reporting channel
5. Not fix anything — findings only; fixes are a separate ingest-style pass

This keeps the lint pass bounded and reviewable rather than a sprawling auto-edit session.

### Phase 3 — `wiki-lint` skill (future)

Once Phase 2 has a stable prompt pattern, encapsulate it as a `wiki-lint` skill parallel to `wiki-done`. This gives it a stable trigger, consistent behaviour across wikis, and a home in the command surface.

---

## Relationship to the command surface

The 040 document (`040-wiki-command-surface.md` in the design lab, or the equivalent in LLM-Wiki) defines the command families. Lint belongs in:

```
wiki-health    → structural + navigational checks (script)
wiki-lint      → semantic + content quality checks (LLM)
```

These are complementary, not competing. `wiki-health` runs every maintenance cycle. `wiki-lint` runs on demand or on a slow schedule.

---

## What this does NOT include

- Auto-repair of lint findings. Lint produces a report; fixing is a separate deliberate pass.
- Full re-read of all wiki pages on every run. Sample-based or triggered approaches are more practical at scale.
- Replacement of `wiki-health`. Structural checks remain script-driven; lint is the semantic layer on top.

---

## Open questions

- What is the right volume threshold for triggering a lint pass? (100 sources? 50 concept pages? Growth rate?)
- Should lint findings be written to `wiki/log.md`, to a dedicated `wiki/lint-log.md`, or both?
- Should Phase 1 `wiki-health` extensions land in the `wiki-health` script in `scripts/`, or as a separate `wiki-lint-structural` script?
- For the IONOS wiki specifically: should the overview map be generated/verified programmatically from the list of sources and concepts, or remain hand-maintained with lint as the safety net?
