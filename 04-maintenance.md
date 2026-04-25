# 04 — Maintenance

## What maintenance is

Maintenance is the ongoing process of keeping the wiki current as its source material changes. It runs on a schedule — not manually, not on a whim.

A wiki that is never maintained is a wiki that goes stale. Sources get updated. New documents are created. Old claims become wrong. The maintenance job is what prevents the wiki from becoming a liability instead of an asset.

## The four-step procedure

Every maintenance run executes these four steps in order.

### Step 1 — Coverage check

Run your coverage tool to identify files in your standing source corpus that are not yet reflected in the wiki.

If you have a coverage tool set up (see `05-cron-spec.md` for notes on this), run it and review the output. Files flagged as `[uncovered]` or `[provenance-only]` are gaps.

If you do not have a dedicated coverage tool, you can approximate this by listing files in the source folder and cross-referencing against the `provenance:` fields in your `wiki/sources/` pages.

The coverage check catches **new files** and **files never ingested**. It does not catch **changed files** — that is what Step 2 is for.

### Step 2 — Detect changed and new source files

This is the step most naive implementations skip, and it is the most important one for a living document tree.

Run:
```bash
git -C /path/to/your/workspace log --since="26 hours ago" \
  --name-only --pretty=format: -- path/to/your/source/corpus/ \
  | sort -u | grep -v '^$'
```

This gives you an exact, deduplicated list of files that were **added or modified** since the last maintenance run.

**Why 26 hours?** The maintenance job runs daily. Using 26 hours instead of 24 gives a 2-hour overlap buffer, so if a run is slightly delayed it does not miss files that changed in the gap.

**Critical:** treat updated files the same as new files. A document that was ingested three months ago and has since been significantly revised needs to be re-ingested. The source page should be updated, not left pointing at stale content.

If the git log returns nothing, cross-reference Step 1 gaps before concluding there is nothing to do.

**This requires git.** Your source corpus must be tracked in a git repository. If it is not, use `find -newer [last-run-sentinel-file] -type f` as an alternative, updating the sentinel file at the end of each run.

### Step 3 — Ingest and update

For each candidate from Steps 1 and 2:

1. Read the source document
2. Decide: is this worth ingesting? (new knowledge, updated knowledge, worth keeping long-term)
3. If yes: create or update the source page in `wiki/sources/`
4. Create or update affected concept pages in `wiki/concepts/`
5. Update index pages (`wiki/sources.md`, `wiki/concepts.md`) if new pages were created
6. Update map pages if the structure or discoverability changed
7. Append to `wiki/log.md`

If the source is not worth ingesting, log a deliberate no-op review for it:
```markdown
## YYYY-MM-DD — No-op review

Reviewed: `path/to/file.md`
Decision: skip — [reason]
```

This makes it clear that the file was seen and consciously skipped, rather than missed.

### Step 4 — Semantic index refresh

If any wiki content was added or updated in this run, refresh the semantic index:
```bash
qmd embed --source /path/to/wiki/
```

Skip this on genuine no-op runs.

## Ingest decision rules

**Ingest when:**
- The content is durable (likely to remain true and useful for months, not days)
- The content is reusable (other queries or sessions would benefit from having it compiled)
- The content answers a question that would otherwise require re-reading a long source document

**Skip when:**
- The content is transient (status updates, meeting notes without lasting conclusions)
- The content is already well-represented in existing concept pages
- The content is a raw operational log without durable insights

**Re-ingest when:**
- A previously ingested source has changed materially
- The existing source page is now inaccurate or incomplete
- New connections have emerged that the original ingest missed

## Logging discipline

Every run — including no-ops — appends an entry to `wiki/log.md`.

**Format:**
```markdown
## YYYY-MM-DD — [Ingest / No-op / Fetch / Error]

Summary of what happened. For ingest runs: list what was created or updated.
For no-op runs: confirm what was checked and why nothing was done.
For error runs: describe what failed and what was left incomplete.
```

Do not edit past log entries. The log is an audit trail.

## Wiki command tools

Three reusable CLI tools are bundled in the `scripts/` folder of this document set. Copy them to your `$PATH` or run them directly with `./scripts/wiki-ingest --wiki ...`. They produce structured output the agent acts on — they handle the mechanical detection work so the agent focuses on judgement and writing.

### `wiki-ingest`

Detects ingest candidates and outputs a structured task for the agent.

```bash
# Detect changed files in last 26 hours + coverage gaps
wiki-ingest --wiki /path/to/YourWiki

# Changed files in last N hours
wiki-ingest --wiki /path/to/YourWiki --since 48

# Ingest a specific file
wiki-ingest --wiki /path/to/YourWiki --source /path/to/file.md

# Full coverage gap scan (no git filter)
wiki-ingest --wiki /path/to/YourWiki --all
```

Exit 0 = candidates found and listed. Exit 1 = nothing to do.

### `wiki-file-back`

Prepares a structured task for filing a durable answer or output back into the wiki.

```bash
# File a content file back as a concept page
wiki-file-back --wiki /path/to/YourWiki --content answer.md --type concept --title "How X works"

# File back from stdin as a synthesis
echo "content" | wiki-file-back --wiki /path/to/YourWiki --stdin --type synthesis

# Types: concept | synthesis | output
#   concept    → wiki/concepts/<slug>.md
#   synthesis  → wiki/syntheses/<date>-<slug>.md
#   output     → outputs/<date>-<slug>.md
```

### `wiki-health`

Runs a structural health check and outputs findings for the agent to act on.

```bash
wiki-health --wiki /path/to/YourWiki
wiki-health --wiki /path/to/YourWiki --json   # machine-readable
```

Checks:
- `[provenance]` — source pages missing `provenance:` frontmatter
- `[orphan]` — pages not listed in their index files
- `[stale-log]` — `wiki/log.md` not updated in >48h
- `[empty-inbox]` — unprocessed files in `raw/inbox/`
- `[broken-link]` — wikilinks pointing to non-existent pages
- `[no-concepts]` / `[no-sources]` — missing core directories

Exit 0 = healthy or warnings only. Exit 2 = errors found.

### Integration with maintenance cron

These tools can be called from the maintenance cron prompt to replace the manual detection steps:

```
STEP 1 — Run: wiki-health --wiki /path/to/YourWiki
         Fix any ERRORs immediately. Note WARNings for the ingest pass.

STEP 2 — Run: wiki-ingest --wiki /path/to/YourWiki
         Act on the listed candidates.
```

This is optional — the git log approach in Step 2 of the four-step procedure works fine without these tools. The tools add convenience and a more structured output format.

## What maintenance is NOT

- **It is not a full re-read of all sources.** You check what changed, not everything.
- **It is not a redesign session.** If you notice structural problems, log them and handle them separately.
- **It is not optional.** A wiki without maintenance is a wiki on its way to being useless.

## Failure modes to avoid

| Failure | Effect | Prevention |
|---------|--------|------------|
| Only checking new files, not changed files | Updated sources produce stale wiki pages | Always run the git diff step |
| Ingesting everything regardless of value | Wiki fills with noise, retrieval degrades | Apply ingest decision rules |
| Skipping the log | No audit trail; no way to know what was checked | Log every run, including no-ops |
| Re-ingesting unchanged files | Wasted compute, unnecessary log noise | Anchor the git diff to the last run window |
| Not refreshing the semantic index | Search returns stale results | Run embed after every real ingest |
