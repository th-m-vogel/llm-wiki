# 02 — Architecture

## Top-level folder layout

```
YourWiki/
  README.md          ← governing handover document; agents read this first
  WIKI-OPERATIONS.md ← operational procedure for maintenance runs
  raw/               ← source material; strictly read-only after capture
  wiki/              ← compiled knowledge; everything the agent writes
  outputs/           ← generated artefacts and reusable answers
  templates/         ← canonical page skeletons
```

The two most important rules:
1. **`raw/` is read-only.** Nothing compiled lives there. Nothing raw lives in `wiki/`.
2. **`wiki/` is what you read when you need to know something.** It is the compiled, curated, retrievable layer.

## raw/ layout

```
raw/
  inbox/             ← new captures land here before processing
  ingested/          ← processed sources, moved here after ingest
  fetch-sources/     ← declaration files for remotely fetched sources (optional)
```

Sources enter through `inbox/`, get ingested into `wiki/`, then move to `ingested/`. This gives you a clean audit trail of what has been processed.

`fetch-sources/` is optional — use it when you have external sources (web pages, APIs, documentation sites) that the agent fetches on a schedule. Each file declares a URL, a fetch method, a refresh interval, and the ingest target.

## Setting up fetch sources

Fetch sources are for knowledge that lives outside your local files — public documentation, web pages, APIs. Instead of manually copying content in, you declare a fetch source and the maintenance agent retrieves and ingests it on a schedule.

Create one markdown file per source in `raw/fetch-sources/`. Each file is a small declaration:

```markdown
---
name: globex-product-docs
url: https://docs.globex.example/product
fetch_method: web
refresh: weekly
last_fetched: 2026-04-25
last_status: ok
status: active
ingest_target: wiki/sources/product-docs.md
---

# Product Docs — Fetch Source

Fetches the main product documentation page and ingests a structured summary.
```

**Fields:**

| Field | Values | Meaning |
|-------|--------|---------|
| `fetch_method` | `web` or `mcp` | `web` = use web_fetch; `mcp` = use an MCP tool if available |
| `refresh` | `daily`, `weekly`, `monthly` | How often to re-fetch |
| `last_fetched` | ISO date | Updated by the agent after each successful fetch |
| `last_status` | `ok`, `error`, `skipped` | Updated by the agent |
| `status` | `active`, `paused`, `pending` | `paused` and `pending` are silently skipped during maintenance |
| `ingest_target` | path in wiki/ | Where the compiled page lives |

During each maintenance run, the agent reads all `status: active` fetch-source files, checks whether the source is due for refresh based on `refresh` and `last_fetched`, fetches if due, and ingests the result. The declaration file is updated with the new `last_fetched` and `last_status`.

Start with zero fetch sources. Add them as you identify external knowledge worth tracking.

## wiki/ layout

```
wiki/
  index.md           ← top-level entry point; links to everything
  sources.md         ← index of all source pages
  concepts.md        ← index of all concept pages
  log.md             ← append-only maintenance log
  sources/           ← one page per ingested source
  concepts/          ← one page per durable concept
  maps/              ← overview and navigation maps
  decisions/         ← significant architectural or workflow decisions
  syntheses/         ← cross-source analysis and coverage pages
```

## Page types

### Source page (`wiki/sources/`)
Records what was ingested from a specific source document.

**Must contain:**
- `provenance:` frontmatter field — where this came from (file path, URL, or identifier)
- `ingested:` frontmatter field — date of ingest
- A summary of the source's content
- Links to the concept pages it contributed to

**Naming:** `YYYY-MM-DD-descriptive-slug.md`

Example:
```markdown
---
provenance: 02_projects/notes/products/globex-compute-platform.md
ingested: 2026-04-25
---
# Globex Compute Platform — Source

Summary of what this document contains...

## Key facts captured
- ...

## Contributed to
- [[../concepts/globex-product-a]]
- [[../concepts/product-compatibility]]
```

### Concept page (`wiki/concepts/`)
A compiled, synthesised explanation of a durable idea, system, or entity. Not tied to a single source — may draw from many.

**Must contain:**
- A clear definition or description
- Links back to the source pages it was built from
- Cross-links to related concepts

**Naming:** `descriptive-slug.md` (no date prefix — concepts are living documents)

### Map page (`wiki/maps/`)
Navigation overview for a cluster of related pages. Helps the agent orient quickly without reading everything.

### Decision page (`wiki/decisions/`)
Records a significant architectural or workflow choice: what was decided, why, and what alternatives were considered. Dated. Immutable after the fact (append notes, don't rewrite history).

### Log (`wiki/log.md`)
Append-only. Every maintenance run appends one entry:
- date and run type (ingest / no-op / fetch)
- what changed, or why it was a no-op
- files affected

This is your audit trail. Never edit past entries.

## Provenance conventions

Every source page must have a `provenance:` field. Use the most specific stable identifier available:

| Source type | Provenance value |
|-------------|-----------------|
| Local file | Relative path from vault root |
| Web page | Full URL |
| API / MCP | Service name + endpoint or resource identifier |
| Human input | `human-input:YYYY-MM-DD` |

When a source is updated, the source page is updated (not replaced) and the change is logged.

## What does NOT go in the wiki

- Raw transcripts or chat logs
- Temporary notes or drafts (use inbox for those)
- Duplicates of source material (summarise, don't copy)
- Opinions or speculation without a source (unless clearly marked as such)
- Anything you wouldn't want to retrieve six months from now and treat as fact

## Linking conventions

Use relative markdown links between wiki pages. If your vault uses Obsidian, use `[[wikilinks]]`. Consistency matters more than which format — pick one and stick to it.

Cross-link liberally. The value of a concept page grows with every inbound link from a source page. If you create a concept but never link to it from anywhere, it is orphaned and will not be found.
