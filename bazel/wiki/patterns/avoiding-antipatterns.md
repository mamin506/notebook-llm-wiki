---
title: "BUILD File Anti-Patterns to Avoid"
category: "patterns"
level: "intermediate"
status: "growing"
sources: ["BUILD Style Guide.md"]
tags: ["anti-patterns", "maintainability", "best-practices"]
graph-group: "patterns"
related: ["[[patterns/build-file-style]]", "[[patterns/dependency-management]]"]
last_updated: "2026-07-19"
---

# BUILD File Anti-Patterns to Avoid

## 1. List Comprehensions

**Problem:** Using list comprehensions at the top level to generate multiple targets.

**Why avoid it:**
- Reduces maintainability (hard for humans and tools to update)
- Reduces discoverability (no named targets, hard to find rules by name)
- Makes automated changes impossible

**Anti-Pattern:**
```starlark
# DON'T: List comprehension generates tests
[[java_test(
    name = "test_%s_%s" % (backend, count),
    srcs = [...],
    deps = [...],
) for backend in ["fake", "mock"]
] for count in [1, 10]
]
```

**Better Approach:**
Define a macro and invoke it explicitly for each target:

```starlark
# DO: Macro + explicit calls
def my_java_test(name, **kwargs):
    java_test(name = name, **kwargs)

my_java_test(name = "test_fake_1", ...)
my_java_test(name = "test_fake_10", ...)
my_java_test(name = "test_mock_1", ...)
my_java_test(name = "test_mock_10", ...)
```

**Benefits:**
- Each target is visible and discoverable
- Tools can find and modify individual targets
- Easier to understand for humans

## 2. Recursive Globs

**Problem:** Using `glob(["**/*.java"])` to match files recursively.

**Why avoid it:**
- Makes BUILD files difficult to reason about (skips subdirectories with BUILD files)
- Less efficient than explicit per-directory BUILD files
- Reduces build cache effectiveness and parallelism

**Anti-Pattern:**
```starlark
# DON'T: Recursive glob
java_library(
    name = "lib",
    srcs = glob(["**/*.java"]),
)
```

**Better Approach:**
Create a BUILD file in each directory with explicit dependencies:

```starlark
# src/main/BUILD
java_library(
    name = "main",
    srcs = glob(["*.java"]),
    deps = ["//src/util"],
)

# src/util/BUILD
java_library(
    name = "util",
    srcs = glob(["*.java"]),
)
```

**Benefits:**
- Enables better remote caching
- Improves build parallelism
- Clear dependency graph between directories

## 3. Non-Recursive Globs

**Status:** Generally acceptable for source files within a package.

```starlark
# OK: Non-recursive glob
java_library(
    name = "lib",
    srcs = glob(["*.java"]),
)
```

## 4. Empty vs. No Match

Use explicit empty lists instead of globs that match nothing:

```starlark
# GOOD: Explicit empty list
cc_library(
    name = "lib",
    srcs = [],  # No source files
)

# BAD: Error-prone glob
cc_library(
    name = "lib",
    srcs = glob(["*.nonexistent"]),  # Confusing intent
)
```

## 5. Limiting .bzl Exports

**Principle:** Each `.bzl` file should export only the symbols intended for use.

**Why:** 
- Reduces rebuild burden (changes to unused symbols still trigger rebuilds)
- Keeps files focused and maintainable
- Prevents "broad libraries" of tangled utilities

**Good Practice:**
```starlark
# util.bzl - Single focused responsibility
def my_rule(name, **kwargs):
    ...

# Only export what's needed
EXPORTED_SYMBOLS = ["my_rule"]
```

**Alternative:** Split into multiple `.bzl` files, each with a clear purpose.

---

See also: [[patterns/build-file-style]], [[patterns/dependency-management]]
