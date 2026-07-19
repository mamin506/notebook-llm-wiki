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

---

## [2026-07-19] architecture | Edge-marginalization strategy for legacy/recommended knowledge

**Principle:** Rather than delete outdated knowledge or treat it equally to modern practices, create a deliberate asymmetry in the wiki:

**Recommended Pages (Rich):**
- Full content with multiple examples
- Cross-linked with many related concepts
- Tagged #recommended, #current-best-practice
- Can grow and expand indefinitely
- Examples: [[concepts/fundamentals/build.bazel]], [[concepts/fundamentals/module.bazel]]

**Legacy Pages (Sparse):**
- Minimal content (one or two sentences)
- Only link back to recommended alternative
- Tagged #legacy, #historical
- Remain stable, rarely edited
- Examples: [[reference/build-legacy]], [[reference/workspace-legacy]]

**Result:** Legacy knowledge is preserved (not deleted) but gently pushed to the edges. Users are guided toward modern practices by the wiki's own structure.

**Implemented Old/New Pairs:**
1. BUILD ↦ BUILD.bazel (file naming)
2. WORKSPACE ↦ MODULE.bazel (project configuration)
3. (Future) http_archive URLs ↦ Bazel modules + registry
4. (Future) --cpu flags ↦ Platforms API
5. (Future) Old macro patterns ↦ Modern best practices

**Benefits:**
- Maintains historical knowledge without pollution
- Reduces maintenance burden (legacy pages are stable)
- Guides learning in right direction through structure
- Future-proofs wiki as Bazel evolves
- Supports users stuck on older versions while encouraging upgrade

**Maintenance Principle:**
- When Bazel deprecates something: create recommended page, then minimal legacy page
- Enhance recommended pages over time
- Legacy pages link ONLY to recommended alternatives
- Never link legacy pages to other concepts (keeps them isolated)

**Created 4 pages with this strategy:**
- [[concepts/fundamentals/build.bazel]] (recommended, rich)
- [[reference/build-legacy]] (legacy, sparse)
- [[concepts/fundamentals/module.bazel]] (recommended, rich)
- [[reference/workspace-legacy]] (legacy, sparse)

This approach ensures the wiki remains useful to learners at all levels while naturally pulling users toward best practices.
