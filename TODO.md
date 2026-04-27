# TODO

Tracked gaps and planned work on the llm-wiki pattern. Add items freely; mark done with `[x]` and a date.

This repo is system-agnostic - implementations exist on Claude Code, OpenClaw/Codex, and others. Notes here should remain generic unless explicitly calling out a platform-specific concern.

---

## Operations

- [x] **Lint operation — Phase 1** — 2026-04-27. Extended `wiki-health` with four new `[WARN]` checks: `[stub-map]` (navigational staleness), `[unmapped]` (coverage drift across all page types), `[quality-floor]` (concept stub detection), `[url-reachable]` (fetch-source URL reachability). Validated on two production wikis. See `04-maintenance.md` for updated check list.
- [x] **Lint operation — Phase 2** — 2026-04-27. LLM-driven semantic lint pass implemented. Skill at `skills/wiki-lint/SKILL.md`. Documented in `04-maintenance.md`. Validated on two production wikis (IONOS Knowledgebase and Marvin Knowledgebase). Findings go to `wiki/lint-log.md`; structural fixes applied immediately; semantic rewrites noted only.

- [x] **Write-back operation** — 2026-04-27. Documented in `04-maintenance.md` (§ Write-back). Skill at `skills/wiki-done/SKILL.md`. Treat conversation as a source; session source page + same tier ingest rules; trigger via explicit closing signal.

## Templates

- [x] **fetch-source.md** — 2026-04-27. Reviewed `templates/fetch-source.md`: Pattern B (url-date existence check) is documented clearly and unambiguously. No changes needed.

## Design notes (not bugs, worth revisiting)

- [x] **Fetch sources as a pattern extension** — 2026-04-27. Already documented in `02-architecture.md` as a first-class layer with full explanation of `raw/fetch-sources/`, detection patterns, and refresh loop. No changes needed.
