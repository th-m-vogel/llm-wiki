# TODO

Tracked gaps and planned work on the llm-wiki pattern. Add items freely; mark done with `[x]` and a date.

This repo is system-agnostic — implementations exist on Claude Code, OpenClaw/Codex, and others. Notes here should remain generic unless explicitly calling out a platform-specific concern.

---

## Operations

- [ ] **Lint operation** — formalize a semantic wiki health pass (LLM-driven, not script-driven). Target: contradictions between pages, stale claims superseded by newer sources, concepts mentioned without their own page, missing cross-references. Distinct from `wiki-health` which only checks structural integrity. Document in `04-maintenance.md` as Step 4 (currently a no-op stub).

- [ ] **Write-back operation** — formalize the query → file-back loop so that durable knowledge generated in conversation compounds into the wiki rather than disappearing into chat history. Design notes:

  **Core idea: treat the conversation as a source.** At the end of a substantive session, the agent creates a dated session source page (`wiki/sources/YYYY-MM-DD-session-topic.md`, `provenance: conversation YYYY-MM-DD`) and runs the same ingest logic as for any other source — same tier rules, same output types. Write-back becomes a special case of ingest, not a separate operation.

  **Trigger options (in order of reliability):**
  1. *Explicit closing signal* — the human sends a single consistent signal when done (a word, a command, whatever the system supports). The agent is instructed to treat it as a session-end trigger and run the sweep before closing. Minimum viable human discipline: one action, always the same.
  2. *Start-of-session sweep* — at the top of each new session, the agent checks `wiki/log.md`; if no session source page exists since the last ingest, it sweeps what changed in the corpus. Avoids the "session ends abruptly" failure mode; trades it for "previous session context is gone" — manageable because the wiki itself is the context.
  3. *Periodic cron sweep* — the maintenance cron checks whether the corpus changed since the last log entry; if so, flags an unswept session. Safety net, not primary trigger.

  **Action:** add a `## Write-back` section to `04-maintenance.md` with the session source page convention, trigger options, and a worked example.

## Templates

- [ ] **fetch-source.md** — template exists; verify the `url-date` (existence-check) pattern is described clearly enough for an agent to apply it without ambiguity to release-note-style sources.

## Design notes (not bugs, worth revisiting)

- **Fetch sources as a pattern extension** — local standing sources (static corpus) + remote fetch sources (web/API/MCP) emerged from the first practical deployment. Not in the original Karpathy concept. Worth documenting explicitly in `02-architecture.md` as a first-class extension of the raw-sources layer, since the distinction between "files I own" and "content I poll" is a real architectural boundary.
