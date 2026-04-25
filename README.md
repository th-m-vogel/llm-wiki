# LLM-Wiki — Agent-Maintained Knowledge Base

---
> **🤖 Agent — stop here first.**
> If you are an agent setting up a new wiki, do not read further until you have asked the user the questions in the **"Agent: gather context before you start"** section below. It is the fifth section of this document, after "Key concepts".
---

## What this is

A practical, battle-tested pattern for giving an AI agent a persistent, searchable knowledge base it maintains itself.

This folder contains everything needed to implement a working wiki from zero — the concept, the architecture, the bootstrap procedure, maintenance automation, reusable scripts, and template files.

## Who this is for

- Any team member who wants their agent to have durable, structured memory beyond chat history
- Any agent receiving this document set as its starting context — see the agent stop notice at the top of this file before reading further

## Why it works

Agents are good at reading, synthesising, and writing structured markdown. They are bad at remembering things across sessions — chat history is expensive, lossy, and session-bound. A wiki sidesteps all of that: durable knowledge lives in files, not in context windows.

The pattern is simple:
- raw source material comes in from wherever your knowledge lives
- the agent reads it, distils what is worth keeping, and writes compiled pages
- pages are small, linked, and explicitly attributed to their sources
- a maintenance job runs on a schedule and keeps the wiki current as sources change

The result is an agent that wakes up fresh each session, reads a compact wiki, and operates from curated knowledge rather than reconstructing everything from scratch every time.

**The primary purpose of the wiki is to be the agent's information source.** It is built by the agent, for the agent. Human readability is a welcome side effect — not the goal. Design decisions (page size, link density, provenance metadata, index structure) should optimise for what makes it easy for the agent to find and use the right knowledge, not for what looks clean in a UI.

## Key concepts

**Standing source corpus** — the folder(s) of documents your agent treats as its primary knowledge input. Think of it as the raw material pile: notes, reports, specs, anything you have accumulated that contains knowledge worth keeping. Your existing Obsidian vault, or a subfolder of it, is a natural fit — point the agent at it and it will start ingesting from what you already have. The corpus itself is never modified — it is read-only input. As you add or update files in the corpus, the maintenance job picks them up automatically.

**Fetch sources** — external URLs (web pages, documentation sites, APIs) the agent fetches on a schedule to keep the wiki current with information that lives outside your local files. Declared as small YAML-style files so the agent knows what to fetch, how often, and where to put the result. Covered in `02-architecture.md`.

**Ingest** — the act of reading a source document and writing a compiled wiki page from it. Not a copy — a distillation. The agent extracts what is durable and reusable, links it to related concepts, and attributes it back to the source for auditability.

---

## Agent: gather context before you start

**Stop here and ask the user the following questions before reading any further documents or creating any files.** Do not assume defaults. Do not proceed until you have answers.

### Required — ask these first

1. **Wiki name and purpose**
   > "What should this wiki be called, and what is its general topic or scope? (e.g. 'our team's practices', 'my personal research notes', 'product knowledge base')"

2. **Where to create it**
   > "Where should the wiki folder live? Please give me a path. If you use Obsidian, this would typically be inside your vault."

3. **Standing source corpus**
   > "Do you have existing notes or documents you want the wiki to learn from? If yes, where are they? This could be an Obsidian vault folder, a project notes directory, or any folder with markdown files. If you have nothing yet, that's fine — we can start empty."

### Optional — ask after the required ones

4. **Semantic search (qmd)**
   > "Would you like me to set up semantic (meaning-based) search on top of the wiki? This requires the `qmd` tool to be installed. It makes retrieval smarter but adds a setup step and a small overhead on each maintenance run. Yes or no?"

5. **External fetch sources**
   > "Are there any external websites or documentation URLs you want the wiki to monitor and ingest on a schedule? For example, a product docs site or a public API reference. If yes, list them — otherwise we skip this."

6. **Maintenance schedule**
   > "How often should the maintenance job run, and at what time? Daily at 02:00 is a reasonable default. Do you want to change this?"

7. **Notifications**
   > "Where should the maintenance job send a summary when it makes changes? Options: Telegram (give me a chat/topic ID), Slack (channel name), Discord, or silent (no notifications, just log to file)."

Once you have answers, proceed through the documents in order: `01-concept.md` → `02-architecture.md` → `03-bootstrap.md` → `04-maintenance.md` → `05-cron-spec.md`. Refer back to the user's answers whenever the documents ask you to fill in paths, names, or preferences.

**After the wiki is live, write the following to your long-term memory** (MEMORY.md or equivalent):
- The wiki's name and purpose
- The absolute path to the wiki root
- That this wiki is your **primary information source** for its domain — consult it before reconstructing knowledge from scratch in any future session
- The maintenance cron schedule so you know when it runs

This is not optional. A wiki you cannot find from memory in a future session is a wiki that does not exist.

---

## What's in this package

```
LLM-Wiki/
  README.md              ← this file
  01-concept.md          ← the why and the core principles
  02-architecture.md     ← folder layout, page types, fetch sources
  03-bootstrap.md        ← stand up a wiki from zero
  04-maintenance.md      ← ongoing maintenance procedure and command tools
  05-cron-spec.md        ← automation template
  scripts/
    wiki-ingest          ← detect ingest candidates (git diff + coverage scan)
    wiki-file-back       ← file a durable answer or output back into the wiki
    wiki-health          ← structural health check
  templates/
    WIKI-README-template.md       ← template for your new wiki's README
    WIKI-OPERATIONS-template.md   ← template for your new wiki's operations doc
```

**Installing the scripts:** copy the `scripts/` folder to somewhere on your `$PATH`, or run them directly with `./scripts/wiki-ingest --wiki ...`. They are plain bash with no dependencies beyond `git`, `bash`, and standard Unix tools (`find`, `grep`, `stat`).

---

## Prior art

This pattern is a production implementation of Andrej Karpathy's LLM Wiki concept.
Original gist: **https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f**

Karpathy's core observation: LLMs are better used as active *compilers* of knowledge into a structured wiki than as passive retrievers from raw documents. The agent reads sources, distils what is durable, writes and cross-links pages, and maintains the wiki over time. This sidesteps the main weakness of RAG (re-synthesising the same raw content on every query) and produces a knowledge base that is human-readable, auditable, and self-improving.

This document set adapts that concept for team use, with concrete architecture decisions, a bootstrap procedure, and production-ready automation.

## Production implementations

This exact pattern runs in production with two live wikis — one for agent design and operations knowledge, one for company and product knowledge — both maintained by nightly cron jobs using the architecture and conventions described in these documents.
