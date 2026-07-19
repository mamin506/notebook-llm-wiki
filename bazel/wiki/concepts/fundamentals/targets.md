---
title: "Targets: Build Units"
category: "concepts"
level: "fundamentals"
status: "growing"
sources: ["Repositories, workspaces, packages, and targets.md"]
tags: ["core", "target"]
related: ["[[concepts/fundamentals/packages]]", "[[concepts/fundamentals/rules]]", "[[concepts/fundamentals/labels]]"]
last_updated: "2026-07-19"
---

# Targets: The Atomic Units of Bazel Builds

A **target** is the atomic unit of a Bazel build. It's a declared entity in a BUILD file that specifies what can be built and how.

## What is a Target?

Targets are containers of two main types: **files** and **rules**.

### Files (Source & Generated)

**Source files:** Written by humans, checked into the repository.

**Generated files:** Created by rules during the build (also called "derived files" or "output files").

### Rules

A **rule** instance specifies the relationship between:
- Input files (source or generated)
- Output files (generated)
- Build steps to transform inputs to outputs

**Example:**
```starlark
cc_binary(
    name = "my_app",          # Target name
    srcs = ["main.cc"],       # Input source files
    deps = ["//lib:mylib"],   # Dependency targets
    # Outputs: executable file named "my_app"
)
```

## Targets Belong to Packages

Every target belongs to exactly one [[concepts/fundamentals/packages|package]]. A target is declared in a BUILD file and named by its `name` attribute.

**Example:**
- Target: `//src/app:my_app`
  - Package: `//src/app`
  - Target name: `my_app`

## Files vs. Rules

An important principle: **It doesn't matter if a target's input is a source file or a generated file.** This allows easy substitution:
- A complex source file can be replaced by a rule that generates it
- A generated file can be replaced by a source file
- Consumers don't need to know the difference

## Target Graph

Targets form a **Directed Acyclic Graph (DAG)** where edges represent dependencies. Bazel uses this graph to:
- Determine build order
- Optimize incremental builds
- Parallelize compilation
- Detect cycles (if any)

---

See also: [[concepts/fundamentals/packages]], [[concepts/fundamentals/rules]], [[concepts/fundamentals/labels]]
