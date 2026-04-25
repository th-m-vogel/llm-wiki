# 03 — Bootstrap

## What you are doing

Standing up a wiki from zero. By the end of this procedure you will have:
- a folder structure ready to receive knowledge
- a governing README the maintenance agent can use as its handover
- an initial set of pages ingested from your existing source material
- a log entry proving the wiki is live

This is a one-time procedure. Maintenance (ongoing ingestion and upkeep) is covered in `04-maintenance.md`.

## Prerequisites

- A folder inside your vault or workspace to host the wiki (e.g. `YourWiki/`)
- At least one folder of existing source material to ingest from (can be empty — but you need somewhere to point the standing-source step)
- An agent with read/write access to the workspace
- **Git** — the source corpus folder must be tracked in a git repository. The maintenance scripts use `git log` to detect changed files. If your workspace is not already a git repo, run `git init` and make an initial commit before proceeding.

## Step 1 — Create the folder structure

```
YourWiki/
  raw/
    inbox/
    ingested/
  wiki/
    sources/
    concepts/
    maps/
    decisions/
    syntheses/
  outputs/
  templates/
```

Create these as empty folders. Most file systems need at least a placeholder file to track empty directories — a `.gitkeep` or a stub README in each is fine.

## Step 2 — Write the governing README

Create `YourWiki/README.md` using `templates/WIKI-README-template.md` as your starting point. Fill in:
- the wiki's name and scope
- the standing source corpus path
- what is in and out of scope

Keep it short. This README is loaded as context on every maintenance run — every extra sentence costs tokens forever.

## Step 3 — Write the operations document

Create `YourWiki/WIKI-OPERATIONS.md` using `templates/WIKI-OPERATIONS-template.md` as your starting point. Fill in:
- absolute paths to the wiki root and source corpus
- the notification target (or remove the reporting rule if running silent)

This document is what the cron job prompt references as the operational authority. The template already contains the full four-step procedure — adapt it, do not rewrite it from scratch.

## Step 4 — Seed the wiki structure

Create these files manually or ask your agent to create them:

**`wiki/index.md`** — top-level entry point. Start with just a title and a note that this wiki was bootstrapped on [date]. Add links as pages are created.

**`wiki/sources.md`** — index of source pages. Start empty with a header.

**`wiki/concepts.md`** — index of concept pages. Start empty with a header.

**`wiki/log.md`** — maintenance log. Add the first entry manually:
```markdown
## YYYY-MM-DD — Bootstrap

Initial wiki structure created. No sources ingested yet.
```

**`wiki/maps/overview.md`** — high-level navigation map. Describe the intended scope and sub-clusters. Update as the wiki grows.

## Step 5 — First ingest pass

If the bundled scripts are installed, run:
```bash
wiki-ingest --wiki /path/to/YourWiki --all
```
This scans the full source corpus for uncovered files and outputs a structured ingest task for your agent to act on.

Alternatively, give your agent this prompt directly:

```
You are bootstrapping a new knowledge wiki at [path to YourWiki/].
Read README.md and WIKI-OPERATIONS.md first.

Then:
1. Review the standing source corpus at [path to your source folder].
2. For each file or subfolder, decide: is this worth ingesting now?
3. For everything worth ingesting: create a source page in wiki/sources/,
   create or update concept pages in wiki/concepts/, update wiki/sources.md
   and wiki/concepts.md, update wiki/maps/overview.md, append to wiki/log.md.
4. Keep provenance explicit on every source page.
5. When done, report what was ingested and what was skipped and why.
```

Do not try to ingest everything at once. Prioritise the most durable, reusable knowledge first. Everything else will be picked up by maintenance over time.

## Step 6 — Set up semantic search (optional but recommended)

If you have `qmd` available (semantic search tool):

```bash
qmd embed --source [path to YourWiki/wiki/]
```

This indexes all compiled pages for vector search. After any ingest that adds or changes content, re-run this command to keep the index current.

Without qmd, the wiki is still fully functional — you rely on `rg` (ripgrep) for text search and on the index/map pages for navigation.

## Step 7 — Verify

Check that the following are true before calling bootstrap complete:

- [ ] `YourWiki/README.md` exists and describes the wiki's purpose
- [ ] `YourWiki/WIKI-OPERATIONS.md` exists and contains the four-step maintenance procedure
- [ ] `wiki/index.md`, `wiki/sources.md`, `wiki/concepts.md`, `wiki/log.md` all exist
- [ ] At least one source page exists in `wiki/sources/`
- [ ] At least one concept page exists in `wiki/concepts/`
- [ ] `wiki/log.md` has at least one entry
- [ ] All source pages have a `provenance:` field

If all boxes are checked, run a final sanity check:
```bash
wiki-health --wiki /path/to/YourWiki
```
Fix any errors before handing off to the maintenance cron (see `05-cron-spec.md`).

## Step 8 — Write it to memory

The wiki is only useful if future sessions know it exists and know to use it. Write the following to your long-term memory file (MEMORY.md or equivalent) **before closing this session**:

```
## [Wiki name] — Knowledge Wiki
- Path: /absolute/path/to/YourWiki
- Purpose: [one line describing the wiki's domain]
- Primary information source for [domain] — consult before reconstructing knowledge from scratch
- Maintained by cron: [schedule, e.g. daily at 02:00]
- Entry point: YourWiki/wiki/index.md
```

A wiki you cannot find from memory in a future session does not exist.
