---
name: wiki-done
description: Use when the user signals the end of a session. Runs a session sweep to file durable knowledge back into the wiki.
platform: Claude Code (adapt trigger and tool names for other platforms)
---

# Wiki Session Done

The user has signalled the end of a substantive session. Run a session sweep to capture anything worth keeping before closing.

## Platform notes

This skill is written for Claude Code. For other platforms:
- **OpenClaw / Codex**: adapt to equivalent closing-signal mechanism; tool names and paths remain the same
- **Trigger phrase**: replace `/wiki-done` with whatever closing convention fits your platform

Fill in `$WIKI_ROOT` with the absolute path to your wiki before deploying.

---

## Steps

### 1 — Review the conversation for durable knowledge

Scan the conversation for content that is:
- **Durable** — not tied to a specific task instance; useful in future sessions
- **New** — not already in the wiki (check `wiki/index.md` and `wiki/concepts.md`)
- **Generalisable** — applies beyond the immediate question; no user- or customer-specific data

Discard: ephemeral answers, lookups of values already in the wiki, task-specific decisions with no reuse value.

### 2 — Decide: sweep or no-op

If nothing qualifies → log a no-op entry in `wiki/log.md` and stop. No session source page needed.

If anything qualifies → continue.

### 3 — Create a session source page

Create `$WIKI_ROOT/wiki/sources/YYYY-MM-DD-session-<topic-slug>.md` with this frontmatter:

```
---
provenance: conversation YYYY-MM-DD
session_topic: <one-line description>
ingested: YYYY-MM-DD
---
```

Body: a brief summary of what was discussed — decisions made, patterns discovered, design rationale. Not a transcript. Two to six bullet points is usually right.

### 4 — File back qualifying content

For each qualifying item, use `wiki-file-back` or write the page directly:

```bash
$WIKI_ROOT/scripts/wiki-file-back \
  --wiki $WIKI_ROOT \
  --stdin \
  --type <concept|synthesis|output> \
  --title "<title>"
```

Apply the same tier rules as any ingest:
- **Tier 1** — new durable concept or significant update → concept page or update existing
- **Tier 2** — cross-source analysis or design decision → synthesis page
- **Tier 3** — noted but not worth a page → session source page body only

### 5 — Update indices and log

- Add session source page to `wiki/sources.md`
- If new concept/synthesis pages were created: update `wiki/concepts.md` and `wiki/index.md`
- Append to `wiki/log.md`:
  ```
  ## YYYY-MM-DD — Session sweep: <topic>
  <one-line summary of what was filed back>
  ```

### 6 — Done

Confirm to the user what was filed (or that it was a no-op). No further action needed.
