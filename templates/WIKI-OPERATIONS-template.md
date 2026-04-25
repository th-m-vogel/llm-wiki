# Wiki Operations — [Wiki Name]

## Maintenance procedure

Every maintenance run follows four steps in order.

### Step 1 — Health check
```bash
wiki-health --wiki /absolute/path/to/YourWiki
```
Fix any `[ERROR]` findings immediately. Note `[WARN]` findings for the ingest pass.
If the tool is not installed, manually check:
- `wiki/sources/` — all pages have `provenance:` frontmatter
- `wiki/log.md` — last entry is recent
- `raw/inbox/` — no unprocessed files sitting there

### Step 2 — Detect changed and new source files
```bash
wiki-ingest --wiki /absolute/path/to/YourWiki
```
This runs a git diff (last 26h) against the standing source corpus and surfaces coverage gaps.
Treat every listed file as a candidate for ingest or re-ingest — including previously ingested files that have since been updated.

If the tool is not installed, run manually:
```bash
git -C /path/to/workspace log --since="26 hours ago" \
  --name-only --pretty=format: -- path/to/source/corpus/ \
  | sort -u | grep -v '^$'
```

### Step 3 — Ingest and update
For each candidate from Steps 1 and 2:
- Read the source document
- Decide: is this worth ingesting? (durable, reusable, long-term value)
- If yes: create or update the source page in `wiki/sources/` with explicit `provenance:` frontmatter
- Create or update affected concept pages in `wiki/concepts/`
- Update `wiki/sources.md` and `wiki/concepts.md` if new pages were created
- Update map pages if discoverability changed
- Append to `wiki/log.md`
- If skipping: log a deliberate no-op review entry with the reason

### Step 4 — Semantic index refresh
If any wiki content was added or updated:
```bash
qmd embed --source /absolute/path/to/YourWiki/wiki/
```
Skip on genuine no-op runs. Skip entirely if qmd is not installed.

---

## Ingest rules

- `raw/` is strictly read-only — never edit source files after capture
- Every source page must have a `provenance:` frontmatter field
- Distil, do not copy — extract what is durable and reusable, link back to source
- Do not ingest transient content (status updates, drafts without conclusions, kanban boards)
- Re-ingest changed files — an updated source with a stale wiki page is a gap

## Reporting rule

- Genuine no-op → `NO_REPLY`, no notification sent
- Durable updates made → send one concise summary to [notification target]; 2-6 bullets; then `NO_REPLY`
- Failed or blocked → send one concise error summary to same target; then `NO_REPLY`
- Never send more than one message per run

## What to avoid

- Do not treat chat history as a durable source of truth
- Do not add pages without provenance when they are source-derived
- Do not leave useful synthesis only in conversation — file it back with `wiki-file-back`
- Do not edit past log entries — the log is an audit trail
- Do not ingest speculatively — if unsure, log a no-op review and move on
