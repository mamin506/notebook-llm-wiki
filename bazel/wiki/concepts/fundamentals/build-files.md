---
title: "BUILD Files: Declaring Targets"
category: "concepts"
level: "fundamentals"
status: "growing"
sources: ["BUILD files.md"]
tags: ["core", "build-files", "starlark"]
related: ["[[concepts/fundamentals/packages]]", "[[concepts/fundamentals/targets]]", "[[patterns/build-file-style]]"]
last_updated: "2026-07-19"
---

# BUILD Files: Declaring Targets

A **BUILD file** is a configuration file that:
- Defines which targets can be built from source
- Declares target properties (sources, dependencies, outputs)
- Uses the Starlark language (Python-like)

## File Names

- `BUILD` (traditional)
- `BUILD.bazel` (recommended for clarity)

Both are valid and treated identically.

## Structure

BUILD files are evaluated as imperative statements, but most consist only of rule declarations (which can be reordered):

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

BUILD files are restricted to prevent side effects:

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

- Comment each BUILD file explaining its purpose
- Comment public vs. private targets
- Use `load()` to reuse common rules
- Keep BUILD files simple and declarative (see [[patterns/build-file-style]])

---

See also: [[patterns/build-file-style]], [[concepts/fundamentals/packages]], [[concepts/fundamentals/targets]]
