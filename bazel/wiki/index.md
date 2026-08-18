# Bazel Wiki Index

This is the catalog of all pages in the Bazel LLM Wiki. Updated after every ingest.

**Status:** First ingest complete! 7 pages created covering BUILD style, best practices, and tools.

---

## Concepts

### Fundamentals
- [[concepts/fundamentals/packages]] — The unit of code organization (growing)
- [[concepts/fundamentals/packages-vs-modules]] — Clear comparison: packages vs modules (seedling)
- [[concepts/fundamentals/targets]] — Atomic build units (growing)
- [[concepts/fundamentals/build.bazel]] — **[Recommended]** Declaring targets with BUILD.bazel files (growing)
- [[concepts/fundamentals/module.bazel]] — **[Recommended]** Declaring modules and dependencies (growing)
- [[concepts/fundamentals/labels]] — Target identifiers and references (growing)
- [[concepts/fundamentals/dependencies]] — Build graph and dependency management (growing)
- [[concepts/fundamentals/rules]] — **[Recommended]** What rules are, anatomy, and lifecycle (growing)
- [[concepts/fundamentals/bazel-commands]] — **[Beginner]** Essential Bazel commands and workflows (seedling)

### Advanced
- [[concepts/advanced/hermeticity]] — Reproducible builds and isolation (growing)
- [[concepts/advanced/platforms]] — **[Recommended]** Modern platform and toolchain APIs (growing)
- [[concepts/advanced/repository-rules]] — Defining and using repository rules (growing)
- [[concepts/advanced/writing-custom-rules]] — **[Recommended]** Implementing rules in Starlark (growing)
- [[concepts/advanced/module-extensions]] — Module extensions for cross-module dependency resolution (growing)
- [[concepts/advanced/build-systems-landscape]] — Overview of Bazel and other build systems (seedling)
- [[concepts/advanced/client-server-architecture]] — Bazel's long-lived server and architecture (seedling)
- [[concepts/advanced/artifact-vs-task-builds]] — Why Bazel uses artifact-based architecture (seedling)
- [[concepts/advanced/bazel-design-decisions]] — Design tradeoffs and engineering philosophy (seedling)
- [[concepts/advanced/limitations-and-future]] — Where Bazel struggles and future directions (seedling)
(Coming soon: macros, aspects, performance optimization)

---

## Reference

- [[reference/cli-reference]] — Complete Bazel CLI commands and options (growing)
- [[reference/build-options]] — Deep reference for build-time options (growing)
- [[reference/bazelrc-configuration]] — Configuring Bazel with .bazelrc files (growing)
- [[reference/modules-version-selection]] — Module version selection and overrides (growing)
- [[reference/build-conventions]] — BUILD file formatting, conventions, and style (growing)
- [[reference/recommended-rules]] — Quality standards for Bazel rulesets (growing)
- [[reference/repositories-workspaces]] — Repos and workspace organization (growing)
- [[reference/external-dependencies]] — Managing external dependencies and modules (growing)
- [[reference/general-rules]] — Utility rules (alias, config_setting, filegroup, genrule, test_suite) (growing)
- [[reference/remote-execution]] — Remote execution, build caching, and RBE configuration (seedling)
- [[reference/registries]] — Bazel registries and module discovery (seedling)
- [[reference/protocol-buffers]] — Protocol buffer rules and cross-language code generation (seedling)
- [[reference/calling-bazel-from-scripts]] — Scripting Bazel, exit codes, automation (seedling)
- [[reference/platforms-toolchains-rules.md]] — Platform and toolchain rule reference (seedling)
- [[reference/adapting-rules-for-rbe]] — Adapting custom rules for remote execution (seedling)
- [[reference/bazel-vs-cmake]] — Detailed comparison of Bazel and CMake (seedling)
- [[reference/execution-strategies]] — Sandboxing and execution strategies (enhanced with implementations) (growing)
- [[reference/execution-tags-and-caching]] — Tags for controlling execution and caching behavior (growing)
- [[reference/remote-caching-setup]] — Setting up and configuring remote caching backends (growing)
- [[reference/build-command-options]] — Deep reference for all build command options (growing)
- [[reference/extra-actions]] — Extra actions rules (deprecated, use aspects) (seedling)
- [[reference/distributed-builds]] — Remote caching and remote execution at scale (seedling)
- [[reference/build-legacy]] — Traditional `BUILD` filename (legacy, see [[concepts/fundamentals/build.bazel]])
- [[reference/workspace-legacy]] — Traditional `WORKSPACE` approach (legacy, see [[concepts/fundamentals/module.bazel]])
- [[reference/platforms-legacy]] — CPU flags and legacy configuration (deprecated, see [[concepts/advanced/platforms]])

---

## Languages

- [[languages/python]] — Python rules, projects, and best practices (growing)
- [[languages/cpp]] — C++ rules, linking, and toolchains (growing)
- [[languages/shell]] — Shell scripts, sh_binary, sh_library, sh_test (seedling)
- [[languages/objective-c]] — Objective-C/Objective-C++ for iOS and macOS (seedling)

(Coming soon: Java, TypeScript guides)

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
- [[patterns/platform-configuration]] — Setting up platforms for your project (growing)
- [[patterns/monorepo-toolchain-organization]] — Organizing multiple toolchains in heterogeneous monorepos (seedling)
- [[patterns/cmake-to-bazel-migration]] — Step-by-step guide for migrating CMake projects to Bazel (seedling)
- [[patterns/building-for-production]] — Complete pipeline from Bazel build to customer deployment (seedling)

---

## Troubleshooting

- [[troubleshooting/hermeticity-issues]] — Debugging non-hermetic builds (growing)

---

## Experiments

- [[experiments/extending-bazel]] — Custom rules, rulesets, and deep-dive into Bazel extensions (seedling)
- [[experiments/publishing-bazel-rules]] — From writing to publishing: repository structure, CI/CD, documentation (seedling)

---

**Latest Update:** Complete rules documentation with fundamentals and advanced implementation guide
**Pages Created:** 65 total (18 concepts, 27 reference, 9 patterns, 1 tool, 2 troubleshooting, 4 languages, 2 experiments)  
**Coverage:** 
- Concepts: Packages, targets, BUILD.bazel [recommended], MODULE.bazel [recommended], labels, dependencies, hermeticity, platforms [recommended], repository-rules, module-extensions
- Reference: CLI reference, build options, .bazelrc configuration, module version selection, build conventions, recommended rules, repos/workspaces, external deps, legacy files
- Patterns: Build style (DAMP), target naming, dependency management, anti-patterns, visibility, platform configuration
- Troubleshooting: Hermeticity debugging
- Tools: Buildifier
  
**Strategy:** Recommended pages are rich with links and concepts; legacy pages are minimal with only links to recommended alternatives  
**Graph View:** All pages tagged with graph-group for visual organization by category  
**Next Steps:** Language guides (Python, Java, C++), Build rules & rule anatomy, Build workflow/execution model
