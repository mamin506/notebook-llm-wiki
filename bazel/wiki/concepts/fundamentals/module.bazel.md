---
title: "MODULE.bazel (Recommended)"
category: "concepts"
level: "fundamentals"
status: "growing"
sources: ["External dependencies overview.md"]
tags: ["core", "modules", "dependencies", "recommended"]
graph-group: "concepts-fundamentals"
related: ["[[concepts/fundamentals/packages-vs-modules]]", "[[reference/repositories-workspaces]]", "[[reference/external-dependencies]]"]
last_updated: "2026-07-19"
---

# MODULE.bazel (Recommended)

**MODULE.bazel** is the modern, recommended way to declare Bazel projects and their dependencies. It enables deterministic dependency resolution and integrates with the Bazel Central Registry.

## What is MODULE.bazel?

A `MODULE.bazel` file at the workspace root declares:
- Your project as a Bazel module (with name and version)
- Direct dependencies on other Bazel modules
- Metadata about your project

## Basic Structure

```starlark
# MODULE.bazel
module(
    name = "my-project",
    version = "1.0.0",
)

# Direct dependencies
bazel_dep(name = "rules_cc", version = "0.1.1")
bazel_dep(name = "rules_python", version = "0.20.0")
bazel_dep(name = "googletest", version = "1.14.0")
```

## How It Works

1. **Module Resolution** — Bazel reads MODULE.bazel and discovers direct dependencies
2. **Registry Lookup** — Looks up all modules in [Bazel Central Registry](https://registry.bazel.build)
3. **Version Resolution** — Applies [MVS algorithm](https://bazel.build/external/module#version-selection) (deterministic)
4. **Transitive Discovery** — Resolves entire dependency tree automatically
5. **Fetching** — Downloads and caches external modules on-demand

## Benefits Over WORKSPACE

| Aspect | MODULE.bazel | WORKSPACE |
|--------|---------|-----------|
| **Version resolution** | Deterministic (MVS) | Manual, error-prone |
| **Transitive deps** | Automatic | Manual |
| **Registry** | Central (BCR) | External URLs |
| **Diamond deps** | Handled automatically | Conflicts possible |
| **Package managers** | Integrated | Limited |

## Dependency Constraints

Declare version constraints for flexibility:

```starlark
bazel_dep(name = "rules_cc", version = ">=0.1.0")
bazel_dep(name = "rules_python", version = "~0.20.0")  # 0.20.x only
```

## Package Manager Integration

MODULE.bazel enables seamless integration with language-specific package managers:

- **Java/Maven** — via `rules_jvm_external`
- **Python/PyPI** — via `rules_python`
- **Go/Modules** — via `rules_go`
- **Rust/Cargo** — via `rules_rust`

Example:
```starlark
bazel_dep(name = "rules_python", version = "0.20.0")

python = use_extension("@rules_python//python:extensions.bzl", "python")
python.toolchain(python_version = "3.11")
```

## Best Practices

- **Keep MODULE.bazel at workspace root** — Don't nest module files
- **Use specific versions initially** — Add version constraints later
- **List only direct dependencies** — Transitive deps are resolved automatically
- **Prefer registry modules** — Use BCR-listed modules when available
- **Lock file** — Use `MODULE.bazel.lock` for reproducibility (committed to git)

---

**See also:** [[reference/repositories-workspaces]], [[reference/external-dependencies]]

**Legacy:** See [[reference/workspace-legacy]] for the traditional `WORKSPACE` approach (no longer recommended).
