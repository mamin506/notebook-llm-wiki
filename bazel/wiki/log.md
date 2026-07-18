# Bazel Wiki Activity Log

Append-only timeline of all ingest, query, and lint operations on the wiki.

Each entry starts with `## [YYYY-MM-DD] <operation> | <description>` so it's grep-able:
`grep "^## \[" log.md` shows all entries.

---

## Log Format

```
## [YYYY-MM-DD] <operation> | <description>

- Created: [[page1]], [[page2]]
- Updated: [[page3]]
- Key findings: Summary
- [Operation-specific fields]
```

Operations:
- **ingest** — Added new source to wiki
- **query** — Answered a question, possibly creating new page
- **lint** — Ran health check on wiki

---

## Activity

(Awaiting first operations...)

---

**Next:** First ingest will be official Bazel concepts guide.
