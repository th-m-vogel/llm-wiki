# TODO

Tracked gaps and planned work on the llm-wiki pattern. Add items freely; mark done with `[x]` and a date.

This repo is system-agnostic - implementations exist on Claude Code, OpenClaw/Codex, and others. Notes here should remain generic unless explicitly calling out a platform-specific concern.

---

## Operations

- [ ] **Lint operation** — formalize a semantic wiki health pass (LLM-driven, not script-driven). Target: contradictions between pages, stale claims superseded by newer sources, concepts mentioned without their own page, missing cross-references. Distinct from `wiki-health` which only checks structural integrity. Document in `04-maintenance.md` as a future step.

- [x] **Write-back operation** — 2026-04-27. Documented in `04-maintenance.md` (§ Write-back). Skill at `skills/wiki-done/SKILL.md`. Treat conversation as a source; session source page + same tier ingest rules; trigger via explicit closing signal.

## Templates

- [x] **fetch-source.md** — 2026-04-27. Reviewed `templates/fetch-source.md`: Pattern B (url-date existence check) is documented clearly and unambiguously. No changes needed.

## Design notes (not bugs, worth revisiting)

- [x] **Fetch sources as a pattern extension** — 2026-04-27. Already documented in `02-architecture.md` as a first-class layer with full explanation of `raw/fetch-sources/`, detection patterns, and refresh loop. No changes needed.
