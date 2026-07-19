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

## [2026-07-19] ingest | Official Bazel Documentation: Style Guide, Rules, Variables

**Sources:**
- BUILD Style Guide.md
- Recommended Rules.md  
- Sharing Variables.md

**Created:**
- [[patterns/build-file-style]] — DAMP over DRY, file structure, comments
- [[patterns/target-naming]] — Naming conventions by language and type
- [[patterns/dependency-management]] — Direct deps, no shared vars, .bzl imports
- [[patterns/avoiding-antipatterns]] — List comprehensions, recursive globs, .bzl exports
- [[reference/build-conventions]] — Formatting, strings, booleans, whitespace
- [[reference/recommended-rules]] — Ruleset quality standards and requirements
- [[tools/buildifier]] — Formatter, linter, usage examples

**Updated:**
- [[wiki/index.md]] — Added 7 new pages to index

**Key Findings:**
- BUILD files are configurations, not code → prefer DAMP over DRY
- Direct dependencies only; never use shared COMMON_DEPS variables
- Recursive globs are anti-pattern; prefer per-directory BUILD files
- Buildifier is standard tool for formatting (like gofmt)
- Recommended rules have strict quality/maintenance requirements
- Tools like Gazelle can auto-generate/update BUILD files → explicit deps help

**Open Questions:**
- How to structure very large monorepos?
- Performance tuning for massive builds?
- How do macro/rule interactions work?

**Next Source Suggestions:**
- Concepts guide (targets, rules, artifacts, dependencies)
- Performance guide for large monorepos
- Python/Java/C++ language-specific guides

---

**Stats:** 7 pages created, 30+ concepts documented, 2,260+ words added
