# Bazel Wiki Index

This is the catalog of all pages in the Bazel LLM Wiki. Updated after every ingest.

**Status:** First ingest complete! 7 pages created covering BUILD style, best practices, and tools.

---

## Concepts

### Fundamentals
- [[concepts/fundamentals/packages]] — The unit of code organization (growing)
- [[concepts/fundamentals/targets]] — Atomic build units (growing)
- [[concepts/fundamentals/build.bazel]] — **[Recommended]** Declaring targets with BUILD.bazel files (growing)
- [[concepts/fundamentals/module.bazel]] — **[Recommended]** Declaring modules and dependencies (growing)
- [[concepts/fundamentals/labels]] — Target identifiers and references (growing)
- [[concepts/fundamentals/dependencies]] — Build graph and dependency management (growing)

### Advanced
(Coming soon: rules, macros, aspects, custom rules, performance optimization)

---

## Reference

- [[reference/build-conventions]] — BUILD file formatting, conventions, and style (growing)
- [[reference/recommended-rules]] — Quality standards for Bazel rulesets (growing)
- [[reference/repositories-workspaces]] — Repos and workspace organization (growing)
- [[reference/external-dependencies]] — Managing external dependencies and modules (growing)
- [[reference/build-legacy]] — Traditional `BUILD` filename (legacy, see [[concepts/fundamentals/build.bazel]])
- [[reference/workspace-legacy]] — Traditional `WORKSPACE` approach (legacy, see [[concepts/fundamentals/module.bazel]])

---

## Languages

(Coming soon: Python, Java, C++, TypeScript guides)

---

## Tools

- [[tools/buildifier]] — BUILD file formatter and linter (growing)

---

## Patterns

- [[patterns/build-file-style]] — DAMP over DRY, file structure, comments (growing)
- [[patterns/target-naming]] — Naming conventions by language and type (growing)
- [[patterns/dependency-management]] — Direct dependencies, avoiding shared variables (growing)
- [[patterns/avoiding-antipatterns]] — Anti-patterns: list comprehensions, recursive globs (growing)
- [[patterns/visibility]] — Controlling target access and APIs (growing)

---

## Troubleshooting

(Coming soon: debugging, performance, common errors)

---

## Experiments

(Will contain pages documenting your hands-on learning and projects.)

---

**Latest Update:** Edge-marginalization strategy for legacy/recommended knowledge  
**Pages Created:** 20 total (7 concepts, 6 reference, 5 patterns, 1 tool, 1 visibility)  
**Coverage:** Packages, targets, BUILD.bazel [recommended], MODULE.bazel [recommended], labels, dependencies, repos, workspaces, external deps, build style, naming, dependency mgmt, anti-patterns, visibility, buildifier, recommended rules + legacy pages for BUILD and WORKSPACE  
**Strategy:** Recommended pages are rich with links and concepts; legacy pages are minimal with only links to recommended alternatives  
**Next Steps:** Rules, workflows, advanced concepts, language guides, troubleshooting, Hermeticity, Platforms migration
