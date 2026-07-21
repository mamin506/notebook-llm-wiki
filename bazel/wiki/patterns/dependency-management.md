---
title: "Dependency Management in Bazel"
category: "patterns"
level: "fundamentals"
status: "growing"
sources: ["BUILD Style Guide.md", "Sharing Variables.md"]
tags: ["dependencies", "best-practices", "maintainability"]
graph-group: "patterns"
related: ["[[patterns/build-file-style]]", "[[concepts/fundamentals/dependencies]]"]
last_updated: "2026-07-19"
---

# Dependency Management in Bazel

## Direct Dependencies Only

**Rule:** List only *direct* dependencies—those actually used by the target.

Do NOT list transitive dependencies (dependencies of dependencies). Bazel resolves the full dependency graph automatically.

```starlark
# BAD: Includes transitive deps
cc_library(
    name = "lib",
    srcs = ["lib.cc"],
    deps = [
        ":helper",      # Direct
        "//util:log",   # Transitive (via :helper)
    ],
)
```

```starlark
# GOOD: Only direct deps
cc_library(
    name = "lib",
    srcs = ["lib.cc"],
    deps = [":helper"],  # Direct only
)
```

## Avoid Shared Dependency Variables

**Rule:** Don't use variables to list "common" dependencies.

### Why Not?

- Reduces maintainability (changing one variable affects many targets)
- Makes it impossible for tools to update individual target dependencies
- Can lead to unused dependencies in targets

### Anti-Pattern

```starlark
COMMON_DEPS = [
    "//d:e",
    "//x/y:z",
]

cc_library(name = "a",
    srcs = ["a.cc"],
    deps = COMMON_DEPS + [...],
)

cc_library(name = "b",
    srcs = ["b.cc"],
    deps = COMMON_DEPS + [...],
)
```

### Better Approach

```starlark
cc_library(name = "a",
    srcs = ["a.cc"],
    deps = [
        "//d:e",
        "//x/y:z",
    ],
)

cc_library(name = "b",
    srcs = ["b.cc"],
    deps = [
        "//d:e",
        "//x/y:z",
    ],
)
```

The repetition is acceptable because:
1. It's clearer and more maintainable
2. Tools like [[tools/gazelle]] can auto-generate and update dependencies
3. Each target's dependencies are explicit and independently modifiable

## Sharing Variables Across BUILD Files

If you need to share *constants* (not dependencies) across multiple BUILD files:

1. Create a `.bzl` file with the shared variable
2. Use `load()` to import it into BUILD files

**Example:** `//path/to/constants.bzl`
```starlark
COPTS = ["-DVERSION=5"]
PACKAGE_PREFIX = "//mylib"
```

**Usage in BUILD files:**
```starlark
load("//path/to:constants.bzl", "COPTS")

cc_library(
    name = "foo",
    srcs = ["foo.cc"],
    copts = COPTS,
)
```

## Package-Local Dependencies

Order dependencies with package-local first:

```starlark
cc_library(
    name = "lib",
    srcs = ["lib.cc"],
    deps = [
        ":helper_a",    # Local first
        ":helper_b",
        "//util:log",   # External
        "//data:load",
    ],
)
```

---

See also: [[patterns/build-file-style]], [[tools/gazelle]]
