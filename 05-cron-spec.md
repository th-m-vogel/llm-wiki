 # 05 — Cron Spec

## Overview

The maintenance job is a scheduled agent turn that runs the four-step procedure from `04-maintenance.md` automatically. It requires:

- An OpenClaw-compatible gateway (or equivalent agent scheduler)
- A persistent or isolated session for the agent to run in
- Read/write tool access to the wiki and source corpus
- `exec` access for the git diff and optional qmd commands

## Minimal cron job definition

This is a template. Adapt paths, schedule, and routing to your setup.

```json
{
  "name": "your-wiki-maintenance",
  "description": "Daily maintenance for YourWiki: detect changed/new sources, ingest durable knowledge, keep coverage and navigation updated.",
  "enabled": true,
  "schedule": {
    "kind": "cron",
    "expr": "0 2 * * *",
    "tz": "Europe/Berlin"   // change to your local timezone
  },
  "sessionTarget": "session:your-wiki-session",
  "payload": {
    "kind": "agentTurn",
    "model": "claude-sonnet",
    "thinking": "low",
    "timeoutSeconds": 900,
    "toolsAllow": ["read", "write", "edit", "exec", "message"],
    // memory_search / memory_get are OpenClaw-specific — add if your platform supports them
    "message": "<see prompt template below>"
  },
  "delivery": {
    "mode": "none"
  }
}
```

**Schedule notes:**
- `0 2 * * *` = 02:00 daily. Good default for maintenance that may take several minutes.
- Adjust to your preferred window. Avoid scheduling during peak usage hours.
- `timeoutSeconds: 900` = 15 minutes. Increase if your source corpus is large.

## Prompt template

Paste this as the `message` field above, replacing bracketed values:

```
Run [YourWiki name] maintenance.
Use [/absolute/path/to/YourWiki/README.md] as the governing handover
and [/absolute/path/to/YourWiki/WIKI-OPERATIONS.md] as the operational procedure.

STEP 1 — Health check
Run: wiki-health --wiki [/absolute/path/to/YourWiki]
Fix any [ERROR] findings immediately. Note [WARN] findings for the ingest pass.
If wiki-health is not installed, check manually: provenance frontmatter on source
pages, wiki/log.md recency, and raw/inbox/ for unprocessed files.

STEP 2 — Detect changed and new source files
Run: wiki-ingest --wiki [/absolute/path/to/YourWiki] --corpus [/absolute/path/to/source/corpus]
Treat every listed file as a candidate for ingest or re-ingest,
including previously ingested files that have been updated.
The --corpus flag is required to detect stale and uncovered standing-source files.

If wiki-ingest is not installed, run both detection methods and combine results:

Method A — mtime-per-file audit (primary; works even without git history):
  find [/absolute/path/to/source/corpus] -name '*.md' | while read src; do
    match=$(grep -rl "$(basename "$src" .md)" [/absolute/path/to/YourWiki/wiki/sources/] 2>/dev/null | head -1)
    if [ -z "$match" ]; then echo "UNCOVERED: $src"
    else
      [ $(stat -c '%Y' "$src") -gt $(stat -c '%Y' "$match") ] && echo "STALE: $src"
    fi
  done

Method B — git log (catches precise recent changes when history exists):
  git -C [/absolute/path/to/workspace] log --since="26 hours ago" \
    --name-only --pretty=format: -- [relative/path/to/source/corpus/] \
    | sort -u | grep -v '^$'
  If git has no history or exits non-zero, skip silently.

Combine results from both methods (deduplicated). UNCOVERED and STALE files
from Method A, plus git results from Method B, form the final candidate list.

STEP 3 — Ingest and update
For any material worth ingesting (new or updated): create/update source pages
in wiki/sources/, create/update concept pages in wiki/concepts/, update
wiki/sources.md, wiki/concepts.md, relevant map pages, append to wiki/log.md.
Keep provenance explicit on every source page.

STEP 4 — Semantic index refresh
If any wiki content was added or updated in this run, run:
  qmd embed --source [/absolute/path/to/YourWiki/wiki/]
Skip if the run was a genuine no-op or if qmd is not installed.

Reporting rule:
- Genuine no-op with no durable changes → reply exactly NO_REPLY
- Durable updates made → send one concise summary to [your notification target];
  2-6 bullets; then reply exactly NO_REPLY
- Failed or blocked → send one concise error summary to same target;
  then reply exactly NO_REPLY

Do not send more than one message per run.
```

## Coverage and health tooling

The bundled `scripts/wiki-health` covers Step 1 fully: provenance gaps, orphan pages, stale log, inbox backlog, and broken wikilinks. Use it instead of a manual check.

The bundled `scripts/wiki-ingest` covers Step 2: git diff detection and coverage gap scan combined.

Both tools degrade gracefully — if git is unavailable or the wiki structure is incomplete, they report the issue rather than silently producing wrong output.

## Session targeting options

| Option | When to use |
|--------|-------------|
| `session:your-named-session` | Persistent session with accumulated context. Good for wikis that benefit from cross-run memory. |
| `isolated` | Fresh session each run. Safer, cheaper, no context bleed. Requires README/ops doc to be self-contained. |
| `session:agent:main:main` | Main agent session (OpenClaw syntax — adapt to your platform's equivalent). Only if wiki maintenance should run in the same context as your primary agent. |

For most setups: use an isolated or named persistent session dedicated to the wiki. Do not run wiki maintenance in your primary interaction session unless they are the same thing by design.

## Notification routing

The prompt template includes a reporting step that sends a summary when changes are made. Route this to wherever your team gets operational updates:

- **Telegram:** `message tool, channel="telegram", target="<chatId>", threadId=<topicId>`
- **Slack:** `message tool, channel="slack", target="<channel-name>"`
- **Discord:** `message tool, channel="discord", target="<channel-id>"`
- **No notifications:** omit the reporting step entirely and rely on `wiki/log.md`

## Tuning for corpus size

| Corpus size | Recommended timeout | Notes |
|-------------|--------------------|-|
| < 50 files | 300s | Comfortable for most runs |
| 50–200 files | 600s | Default recommendation |
| 200+ files | 900s | May need to split into sub-jobs |

If your corpus is large enough that a single run cannot process all changes, consider splitting maintenance into sub-wikis or prioritising Tier 1 sources in the cron prompt and handling Tier 2+ on a separate, less frequent schedule.

## What to monitor

After the first few runs, check:

1. **Run duration** — if consistently near the timeout, increase it or split the job
2. **Token usage** — if very high, tighten the prompt or reduce the scope per run
3. **Log entries** — confirm the agent is actually appending to `wiki/log.md` each run
4. **Source page provenance** — spot-check that `provenance:` fields are accurate and not hallucinated
5. **No-op rate** — if every run is a no-op, check whether the staleness detection is working correctly: confirm `wiki-ingest` is receiving `--corpus`, and verify mtime audit output by running the script manually
