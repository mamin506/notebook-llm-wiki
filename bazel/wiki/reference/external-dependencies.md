---
title: "External Dependencies"
category: "reference"
level: "intermediate"
status: "growing"
sources: ["External dependencies overview.md"]
tags: ["external-deps", "modules", "registry"]
related: ["[[reference/repositories-workspaces]]"]
last_updated: "2026-07-19"
---

# External Dependencies

**External dependencies** are source files used in your build that aren't in your workspace. Examples:
- Rulesets on GitHub
- Maven/npm packages
- Local directories outside your workspace

## Modern Approach: Bazel Modules

**Bazel modules** are versioned Bazel projects declared in `MODULE.bazel`:

```starlark
# MODULE.bazel
module(name = "my-project", version = "1.0")

bazel_dep(name = "rules_cc", version = "0.1.1")
bazel_dep(name = "rules_python", version = "0.20.0")
```

### How It Works

1. Bazel reads `MODULE.bazel` and discovers direct dependencies
2. Looks up dependencies in **Bazel Central Registry (BCR)**
3. Resolves transitive dependencies using **MVS algorithm** (deterministic)
4. Fetches sources for each module
5. Module extensions can integrate with package managers (Maven, PyPI, npm)

## Benefits

- **Deterministic version resolution** — No conflicts with diamond dependencies
- **Strict dependency visibility** — Only direct dependencies visible
- **Unified ecosystem** — Single source for Bazel rules
- **Package manager integration** — Maven, PyPI, npm all work together

## Legacy Approach: WORKSPACE

Older projects use `WORKSPACE` file instead of `MODULE.bazel`:

```python
# WORKSPACE (legacy)
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

http_archive(
    name = "rules_cc",
    urls = ["https://github.com/bazelbuild/rules_cc/archive/refs/tags/0.1.1.zip"],
    strip_prefix = "rules_cc-0.1.1",
)
```

## Referencing External Repos

Use `@` prefix in labels:

```starlark
deps = [
    "@rules_cc//cc:cc_library",      # From rules_cc module
    "@other_repo//lib:mylib",         # From other_repo
]
```

## Fetching Dependencies

Bazel fetches external repos **on demand** when first referenced. Force pre-fetch:

```bash
bazel fetch //my/app:app
```

---

See also: [[reference/repositories-workspaces]]
