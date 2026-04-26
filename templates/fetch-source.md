---
type: wiki-source
source_type: fetch-source
title: ""
date: YYYY-MM-DD
url: ""
last_fetched: YYYY-MM-DD
document_version: ""
detection_method: version-string | url-date | content-diff | periodic
provenance: ""
status: active
related:
  -
---

# Source Title

## Detection

<!-- REQUIRED: Declare which signal this source uses to determine whether re-ingestion is needed.
     Choose one pattern and describe it concretely. -->

<!-- Pattern A — version-string:
     The page contains an embedded date or version string (e.g. "Last updated: 2026-04-13").
     Detection: fetch the page, extract that string, compare to `document_version` in frontmatter.
     If newer → re-ingest and update `last_fetched` and `document_version`. -->

<!-- Pattern B — url-date (existence check):
     The URL encodes a time period (e.g. /april-2026). A new period = new content.
     Detection: if no source page exists for the current period → fetch and ingest.
     Existence of the source page is the signal; no content comparison needed. -->

<!-- Pattern C — content-diff:
     No version string in the page. Detect changes structurally.
     Detection: fetch the live page, compare the relevant structure (e.g. API names, endpoint URLs,
     section headings) against what is stored in the wiki. Any addition or change is the trigger.
     Document which fields or structures to diff here. -->

<!-- Pattern D — periodic (unconditional):
     Source changes frequently or has no reliable version signal.
     Detection: re-fetch on every maintenance run regardless. Use sparingly. -->

## Summary

## Key points

## Gaps / open questions
