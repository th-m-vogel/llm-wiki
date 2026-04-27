---
name: wiki-lint
description: Use when the user requests a semantic lint pass on the wiki (/wiki-lint or equivalent). Checks for semantic staleness, structural gaps, map coherence, and provenance rot. Distinct from wiki-health (structural/script checks) — this is an LLM-driven quality pass. Do NOT run during routine maintenance; only on explicit request or when volume threshold is reached (default: 10 new sources since last lint pass).
platform: Claude Code (adapt trigger and paths for other platforms)
---

# Wiki Semantic Lint Pass

The user has requested a lint pass. Run a bounded semantic health check and produce a findings report.

> **Findings only — do NOT auto-fix content.** Fix clear structural issues (broken cross-links, map prose drift). Note semantic issues for deliberate follow-up.

## Constants

Fill in before deploying:

```
WIKI_ROOT = /path/to/YourWiki
LINT_LOG  = $WIKI_ROOT/wiki/lint-log.md
QMD_BIN   = /path/to/qmd  # optional, skip if not configured
```

---

## Step 1 — Determine scope

```bash
find $WIKI_ROOT/wiki -name "*.md" | wc -l
```

Check `$LINT_LOG` for the last pass date and page count, then apply the appropriate phase:

| Phase | Wiki size | Scope |
|-------|-----------|-------|
| 1 | < 150 pages | Full sweep: all maps + all concepts + newest 15 sources |
| 2 | 150–400 pages | All maps + concepts modified since last lint + sources modified since last lint + random 10 |
| 3 | 400+ pages | Cluster-scoped: one map cluster per pass, rotate through clusters |

Phase 3: check `$LINT_LOG` to see which cluster was last linted; lint the next one.

## Step 2 — Read all map pages

Read every file in `$WIKI_ROOT/wiki/maps/`. Always — regardless of phase. This is the primary navigational surface.

## Step 3 — Read concept pages

Phase 1: read all files in `$WIKI_ROOT/wiki/concepts/`.

Phase 2+: read only concepts modified since the last lint pass date:
```bash
find $WIKI_ROOT/wiki/concepts -name "*.md" -newer $LINT_LOG
```

## Step 4 — Sample source pages

```bash
# Newest 15 by mtime
find $WIKI_ROOT/wiki/sources -name "*.md" -printf "%T@ %p\n" | sort -rn | head -15 | awk '{print $2}'
```

Phase 2+: also include sources modified since last lint pass.

## Step 5 — Run semantic checks

For each page reviewed, check the following. Note findings — do not rewrite page content.

### Semantic staleness
- Does a concept page state something a newer source contradicts or supersedes?
- Is a concept page thin (one source) when multiple sources now exist that would enrich it?

### Structural gaps — cross-links
- Are two closely related concept pages missing a link to each other?
- Is a named entity (product, team, technology) mentioned in 5+ sources but lacking a concept page?

### Map coherence
- Does each cluster's prose description still match its linked pages?
- Has any cluster grown beyond its stated scope without its prose being updated?

### Provenance rot
- For fetch-source pages: does the summary still reflect what the live source contains?
- Are any source pages pointing at internal documents that may have moved or changed?

## Step 6 — Apply structural fixes

Fix these without waiting for human review:
- Missing cross-links between clearly related concept pages
- Map cluster prose that no longer matches its contents
- Broken wikilinks (wrong slug)

Do NOT auto-rewrite concept page bodies or source summaries — note those in the lint log.

## Step 7 — Write findings to lint-log.md

Append to `$LINT_LOG` (create if it doesn't exist):

```markdown
## YYYY-MM-DD — Lint pass

Pages reviewed: N concept pages, M source pages, all maps
Wiki size at pass: X pages total
Phase: 1 / 2 / 3

### Fixes applied

#### [cross-link] concept-slug
Description of what was missing and what was added.

#### [map-coherence] map-slug — Cluster N
Description of prose drift and correction.

### Noted — no fix yet

#### [semantic-staleness] concept-slug
Description of what is stale and why.

#### [structural-gap] entity-name (no concept page)
Description of the gap.

### No findings
(if nothing found, state this explicitly — a clean pass is valid)
```

## Step 8 — Update log and index

```bash
# Append to wiki/log.md
echo "YYYY-MM-DD — Semantic lint pass: N fixes, M noted. See wiki/lint-log.md." >> $WIKI_ROOT/wiki/log.md

# Refresh semantic index if pages were written
$QMD_BIN embed  # skip if qmd not configured
```

## Step 9 — Confirm wiki-health still clean

```bash
wiki-health --wiki $WIKI_ROOT
```

## Step 10 — Report to user

Brief summary: pages reviewed, fixes applied, items noted. Two to four lines.
