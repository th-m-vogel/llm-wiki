# 01 — Concept

## The problem with agent memory

An AI agent's default memory is its context window. That context is:
- **Session-bound** — gone when the session ends
- **Expensive** — every token of history costs money on every call
- **Lossy** — older messages get truncated or dropped under pressure
- **Unstructured** — a flat chronological transcript is a terrible retrieval surface

The standard workaround — "just put everything in the system prompt" — breaks down quickly. System prompts grow unbounded, retrieval degrades, and you end up paying to re-process the same background every single call.

## The insight

Files are free. Reads are cheap. Markdown is structured enough for an agent to navigate, and flat enough that no special tooling is required to write it.

If you give an agent a folder of well-organised markdown files and a clear convention for how they relate to each other, the agent can:
- read the index to orient itself
- follow links to the relevant pages
- find what it needs without loading everything
- write new pages and update existing ones as knowledge accumulates

This is exactly what a wiki is. The agent becomes the wiki's maintainer.

## Why agents are good at this

Agents trained on large corpora have seen enormous amounts of structured text — documentation, encyclopaedias, technical writing. They understand:
- what a concept page looks like
- how to write a useful summary
- what provenance means and why it matters
- how to link related ideas without duplicating them

An agent can read a dense source document, extract what is durable and reusable, write a clean summary page, and link it into the existing structure — in one pass, reliably, at scale.

## The Karpathy observation

Andrej Karpathy has written about the idea that LLMs work well as personal knowledge management systems — they can read sources, synthesise knowledge, and maintain structured notes in a way that integrates naturally with how agents already operate. The wiki pattern takes this observation seriously and builds a production-grade implementation around it.

The key insight worth preserving: **optimise for AI retrieval first, human navigation second.** The pages should be written in a way that makes it easy for the agent to find and use the right knowledge, not primarily in a way that looks pretty in a UI.

## Core principles

**Small, linked notes over giant dumps.**
A 200-line page that covers one concept well is worth ten times a 2000-line dump that covers everything badly. Links are cheap. Splitting is easy. Merging is hard.

**Explicit provenance.**
Every compiled page should know where it came from. When sources change, you need to know which pages are affected. When claims are questioned, you need to be able to verify them. Provenance is not bureaucracy — it is how the wiki stays trustworthy over time.

**Raw sources stay raw.**
There is a strict separation between source material (captured as-is) and compiled knowledge (written by the agent). Raw sources are never edited after capture. Compiled pages are never treated as sources. This separation keeps the wiki auditable.

**Maintenance is a first-class workflow.**
A wiki that is never updated is a wiki that goes stale. Maintenance — reviewing new sources, re-ingesting changed documents, pruning outdated claims — is not optional cleanup. It is the job. It runs on a schedule, not on a whim.

**No-ops are logged.**
Even when a maintenance pass finds nothing to do, it should record that it ran and checked. This makes it possible to audit the maintenance history and distinguish "nothing happened" from "nobody looked."
