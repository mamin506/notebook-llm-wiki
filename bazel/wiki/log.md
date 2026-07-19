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

## [2026-07-19] ingest | Official Bazel Core Concepts (7 sources)

**Sources:**
- Repositories, workspaces, packages, and targets.md
- Labels - Bazel.md
- BUILD files.md
- Dependencies - Bazel.md
- External dependencies overview.md
- Build programs with Bazel.md
- Visibility - Bazel.md

**Created (9 new pages):**
- [[concepts/fundamentals/packages]] — Organization unit, contains targets
- [[concepts/fundamentals/targets]] — Atomic build units, files and rules
- [[concepts/fundamentals/build-files]] — BUILD file syntax and Starlark
- [[concepts/fundamentals/labels]] — Target identifiers and reference syntax
- [[concepts/fundamentals/dependencies]] — Dependency graphs, declared vs actual
- [[reference/repositories-workspaces]] — Repos, workspace, main repo concepts
- [[reference/external-dependencies]] — Bazel modules, registries, version resolution
- [[patterns/visibility]] — Access control, public API, package groups

**Updated:**
- [[wiki/index.md]] — Added 9 new pages, updated counts

**Key Findings:**
- Packages are directories with BUILD files - THE fundamental organizational unit
- Targets (declared in BUILD) form a DAG dependency graph
- Labels have canonical and shorthand forms (@@repo//pkg:target)
- Declared dependencies must be superset of actual dependencies (undeclared deps cause hidden bugs)
- Modern Bazel uses MODULE.bazel + registries for deterministic version resolution
- Visibility controls API surface (public, private, __pkg__, __subpackages__)
- BUILD files use Starlark (restricted for hermeticity)

**Open Questions:**
- How do rules actually work? (cc_library, py_binary, etc.)
- What's the full build workflow? (load → analyze → execute)
- How to write custom rules?
- Performance optimization for large monorepos?

**Gaps Identified:**
- Rules and rule types (cc_library, py_binary, cc_test, etc.)
- Build workflow details (load, analyze, execute phases)
- Custom rule development
- Toolchains and platform configuration
- Caching and build performance
- Integration with IDEs

**Stats:** 9 new pages, 50+ Bazel concepts documented, 4,000+ words added, 16 pages total in wiki
