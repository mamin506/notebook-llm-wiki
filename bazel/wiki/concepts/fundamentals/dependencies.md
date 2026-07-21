---
title: "Dependencies: The Build Graph"
category: "concepts"
level: "fundamentals"
status: "growing"
sources: ["Dependencies    Bazel.md"]
tags: ["core", "dependencies", "dag"]
graph-group: "concepts-fundamentals"
related: ["[[concepts/fundamentals/targets]]", "[[patterns/dependency-management]]"]
last_updated: "2026-07-19"
---

# Dependencies: The Build Graph

A **dependency** means target A needs target B at build or execution time.

Targets form a **Directed Acyclic Graph (DAG)** where edges are dependencies.

## Direct vs. Transitive

- **Direct dependencies:** Targets reachable by a path of length 1
- **Transitive dependencies:** Targets reachable by any length path

**Example:**
```
A depends on B
B depends on C
```

- A's direct dependency: B
- A's transitive dependency: C (via B)

## Declared vs. Actual Dependencies

### Declared Dependencies
What you write in the BUILD file's `deps` attribute.

### Actual Dependencies
What the code actually uses at runtime.

**Key principle:** Declared dependencies must be a *superset* of actual dependencies.

```
Declared ⊇ Actual
```

Failure to declare actual dependencies causes subtle bugs when upstream changes.

## The Undeclared Dependency Problem

**Scenario:**
1. A actually depends on C
2. A only declares dependency on B
3. B happens to depend on C, so the build works
4. B removes its dependency on C
5. A's build now fails (C is missing)

**Solution:** Always declare all direct dependencies.

## Dependency Types in BUILD Files

### `srcs` — Source Inputs
Files consumed directly:

```starlark
cc_library(
    name = "lib",
    srcs = ["lib.cc"],  # Source files
)
```

### `deps` — Compiled Dependencies
Separately-compiled modules providing symbols, headers, libraries:

```starlark
cc_binary(
    name = "app",
    srcs = ["app.cc"],
    deps = ["//lib:mylib"],  # Compiled dependencies
)
```

### `data` — Runtime Data
Data files needed at execution (tests, configs):

```starlark
py_test(
    name = "test",
    srcs = ["test.py"],
    data = ["test_config.txt"],  # Runtime data
)
```

## Best Practices

- Declare only **direct** dependencies (not transitive)
- Use `srcs` for source files, `deps` for targets
- Use `data` for files needed at runtime
- Order dependencies: local first, then external

---

See also: [[patterns/dependency-management]], [[concepts/fundamentals/targets]]
