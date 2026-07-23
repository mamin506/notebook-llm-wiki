---
name: ingest
description: "Use when: ingesting a new source into the Bazel wiki, extracting the key ideas, creating or updating pages, and recording the work."
---

# Ingest

## When to Use
- A new source is added under raw/inbox/
- You need to turn source material into wiki pages
- You want to expand the wiki with new concepts, examples, or references

## Procedure
1. Check raw/inbox/ for pending sources. If the inbox is empty, stop and report that there is nothing new to ingest.
2. Read each pending source from raw/inbox/ and capture the main ideas, examples, contradictions, and gaps.
3. Create or update wiki pages for each significant concept. Prefer updating an existing page when coverage already exists.
4. Add cross-references with wikilinks and keep the frontmatter coherent: title, category, level, status, sources, tags, related, and last_updated.
5. Refresh the index in wiki/index.md and append a concise entry to wiki/log.md with what changed and what remains open.
6. Move each successfully ingested file from its current location in raw/inbox/ to the matching subfolder under raw/processed/, preserving the relative subfolder path (for example, raw/inbox/articles/foo.md -> raw/processed/articles/foo.md). This keeps the queue explicit and the workflow idempotent.
7. Review the result for traceability and usability before stopping.

## Quality Checks
- Each important idea is represented in the wiki with clear content or an update.
- New or modified pages point back to the source and have current metadata.
- Cross-references are present, the index is updated, and the activity log records the ingest.
