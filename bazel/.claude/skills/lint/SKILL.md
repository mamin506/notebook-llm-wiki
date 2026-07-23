---
name: lint
description: "Use when: doing a maintenance pass for the Bazel wiki, checking for stale content, contradictions, orphan pages, missing links, and gaps."
---

# Lint

## When to Use
- You are doing a monthly or periodic health check
- A large ingest batch changed many pages
- You want to improve coherence, freshness, or discoverability

## Procedure
1. Review the wiki for contradictions, stale claims, and outdated metadata.
2. Update or clarify pages, refreshing last_updated and status when content materially changes.
3. Find orphan pages and integrate them, merge them, or remove them if obsolete.
4. Identify gaps that deserve a seedling page or a follow-up source.
5. Verify that cross-references are healthy and record the lint pass in wiki/log.md.

## Quality Checks
- Contradictions are resolved or clearly contextualized.
- Stale pages are updated or flagged for follow-up.
- Orphans and missing coverage are addressed, and the maintenance work is logged.
