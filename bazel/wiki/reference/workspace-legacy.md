---
title: "WORKSPACE / WORKSPACE.bazel (Legacy)"
category: "reference"
level: "intermediate"
status: "stable"
sources: ["External dependencies overview.md"]
tags: ["legacy", "historical", "workspace", "dependencies"]
related: ["[[concepts/fundamentals/module.bazel]]"]
last_updated: "2026-07-19"
---

# WORKSPACE / WORKSPACE.bazel (Legacy)

**WORKSPACE** and **WORKSPACE.bazel** are the traditional way to declare Bazel projects and dependencies. They remain fully supported but are **no longer recommended** for new projects.

## Current Status

- ✅ **Fully supported** — Existing projects continue to work
- ⚠️ **Legacy** — Replaced by MODULE.bazel in modern Bazel
- ➡️ **Migrate to:** [[concepts/fundamentals/module.bazel]]

## Why Not WORKSPACE?

Traditional approach using `WORKSPACE`:

```python
# WORKSPACE (legacy)
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

http_archive(
    name = "rules_cc",
    urls = ["https://github.com/bazelbuild/rules_cc/archive/refs/tags/0.1.1.zip"],
    strip_prefix = "rules_cc-0.1.1",
)
```

**Problems:**
- Manual version management (error-prone)
- Manual transitive dependency resolution
- No central registry
- Diamond dependency conflicts
- Hard to integrate with package managers

## Modern Approach

[[concepts/fundamentals/module.bazel]] solves these problems:

```starlark
# MODULE.bazel (recommended)
module(name = "my-project", version = "1.0.0")

bazel_dep(name = "rules_cc", version = "0.1.1")
```

- Deterministic version resolution
- Automatic transitive dependencies
- Central registry (BCR)
- Seamless package manager integration

## Using WORKSPACE Today

If you're in a codebase still using WORKSPACE:
- Keep using it for now (it's fully supported)
- Plan migration to MODULE.bazel
- Check [Bazel migration guide](https://bazel.build/external/module#migration) for upgrade path

---

**Recommended:** Use [[concepts/fundamentals/module.bazel]] for all new projects. Migrate existing projects when feasible.
