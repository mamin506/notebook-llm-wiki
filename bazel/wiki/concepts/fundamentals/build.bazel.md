---
title: "BUILD.bazel Files (Recommended)"
category: "concepts"
level: "fundamentals"
status: "growing"
sources: ["BUILD files.md"]
tags: ["core", "build-files", "starlark", "recommended"]
graph-group: "concepts-fundamentals"
related: ["[[concepts/fundamentals/packages]]", "[[concepts/fundamentals/targets]]", "[[patterns/build-file-style]]", "[[reference/build-conventions]]"]
last_updated: "2026-07-19"
---

# BUILD.bazel Files (Recommended)

**BUILD.bazel** is the recommended filename for Bazel configuration files. It's a clear, explicit way to declare what targets can be built from source.

## Why BUILD.bazel?

**Clarity:** The `.bazel` extension unambiguously identifies this as a Bazel build file, avoiding confusion with other build systems (Maven, Gradle, etc.) that also use `BUILD`.

**Modern standard:** Bazel's official documentation and new projects standardize on `BUILD.bazel`.

**Future-proof:** Supports better tooling integration and IDE support.

## File Structure

BUILD.bazel files are evaluated as imperative statements, but most consist only of rule declarations (which can be reordered):

```starlark
# Package description comment
package(default_testonly = True)

# Load extensions (rules, macros)
load("//tools:build.bzl", "my_rule")

# Rule declarations (order doesn't matter within this block)
cc_library(
    name = "mylib",
    srcs = ["lib.cc"],
)

cc_binary(
    name = "app",
    srcs = ["app.cc"],
    deps = [":mylib"],
)
```

## Limitations

BUILD.bazel files are restricted to ensure reproducibility:

**NOT allowed:**
- Function definitions
- `for` statements (but list comprehensions OK)
- `if` statements (but `if` expressions OK)
- Arbitrary I/O operations
- `*args` and `**kwargs`

**Reason:** Ensures builds are hermetic and reproducible — dependent only on known inputs.

## Loading Extensions

Use `load()` to import rules and macros from `.bzl` files:

```starlark
load("//tools:defs.bzl", "my_rule", renamed = "other_rule")

my_rule(name = "foo", ...)
```

## Best Practices

- Comment each BUILD.bazel file explaining its package's purpose
- Document which targets are public API vs internal
- Use `load()` to reuse common rules and macros
- Follow [[patterns/build-file-style]] conventions
- Use [[tools/buildifier]] for consistent formatting

---

**See also:** [[patterns/build-file-style]], [[reference/build-conventions]], [[concepts/fundamentals/packages]]

**Legacy:** See [[reference/build-legacy]] for the traditional `BUILD` filename (no longer recommended).
