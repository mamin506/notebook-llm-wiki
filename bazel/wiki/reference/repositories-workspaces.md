---
title: "Repositories & Workspaces"
category: "reference"
level: "fundamentals"
status: "growing"
sources: ["Repositories, workspaces, packages, and targets.md", "External dependencies overview.md"]
tags: ["core", "workspace", "repository"]
graph-group: "reference"
related: ["[[concepts/fundamentals/packages]]", "[[reference/external-dependencies]]"]
last_updated: "2026-07-19"
---

# Repositories & Workspaces

## Repository

A **repository** (repo) is a directory tree with a boundary marker file at its root:
- `MODULE.bazel` — Bazel module (modern)
- `REPO.bazel` — Custom repo marker
- `WORKSPACE` or `WORKSPACE.bazel` — Legacy workspace markers

A repository contains source files organized in nested packages.

## Main Repository

The repository where you run the `bazel` command.

```bash
cd ~/my-project  # This is the main repo
bazel build //app:main
```

The root of the main repository is the **workspace root**.

## Workspace

A **workspace** is the build environment encompassing:
- The **main repository** (where you're working)
- All **external repositories** (dependencies)

All Bazel commands run within a workspace.

### Historical Note
"Workspace" and "repository" have historically been conflated. The term "workspace" was sometimes used to mean "main repository." Modern Bazel uses precise terminology:
- **Repository** — A directory tree with a marker file
- **Workspace** — The environment for all commands in a main repo

## External Repositories

External repos are dependencies fetched from:
- GitHub
- Maven/npm registries
- Local directories
- HTTP URLs

**Declared in:** `MODULE.bazel` (or legacy `WORKSPACE`)

**Example:**
```starlark
# MODULE.bazel
module(name = "my-project", version = "1.0")

bazel_dep(name = "rules_cc", version = "0.1.1")
bazel_dep(name = "rules_python", version = "0.20.0")
```

**Fetched on demand** when referenced in labels:

```starlark
deps = ["@rules_cc//cc:cc_library"]
```

## Repository Layout

```
my-project/              ← Main repo (MODULE.bazel at root)
├── MODULE.bazel
├── WORKSPACE.bazel      ← Can coexist with MODULE.bazel
├── BUILD
├── src/
│   ├── BUILD
│   └── app.cc
└── lib/
    ├── BUILD
    └── lib.cc
```

---

See also: [[concepts/fundamentals/packages]], [[reference/external-dependencies]]
