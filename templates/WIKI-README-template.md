# [Wiki Name]

## What this is

[One paragraph: what domain or scope this wiki covers, and who it is for.]

## What a designated agent should do

1. Read this README
2. Use this wiki as the primary compiled source for [domain] knowledge
3. Maintain it when learning something durable — ingest new sources, update stale pages
4. Run `WIKI-OPERATIONS.md` as the operational procedure for maintenance passes

## Primary entry points

- `wiki/index.md` — top-level index
- `wiki/maps/overview.md` — high-level navigation map
- `wiki/log.md` — maintenance history

## Source model

This wiki ingests from:
- **Standing source corpus:** `[path/to/your/source/folder/]` — primary authoring space; all new documents added here are candidates for ingest
- **Raw inbox:** `raw/inbox/` — staging area for material that arrives outside the standing corpus

`raw/` is strictly read-only after capture. Compiled knowledge lives in `wiki/`, not in `raw/`.

## Scope

[What belongs in this wiki. Be specific enough that an agent can decide whether a new document is in scope without asking.]

## Out of scope

[What should NOT be ingested. Transient files, kanban boards, drafts, etc.]
