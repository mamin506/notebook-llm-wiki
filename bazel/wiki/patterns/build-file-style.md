---
title: "BUILD File Style & Best Practices"
category: "patterns"
level: "fundamentals"
status: "growing"
sources: ["BUILD Style Guide.md"]
tags: ["style", "patterns", "readability"]
related: ["[[reference/build-conventions]]", "[[patterns/target-naming]]", "[[tools/buildifier]]"]
last_updated: "2026-07-19"
---

# BUILD File Style & Best Practices

## DAMP Over DRY

**Key Principle:** Prefer **DAMP** (Descriptive and Meaningful Phrases) over **DRY** (Don't Repeat Yourself) in BUILD files.

Unlike code, BUILD files are *configurations*, not programs. They are:
- Maintained by humans and tools (not tested like code)
- Read far more often than written
- Easier to understand when explicit and clear

**Benefit:** Explicit BUILD files allow automated tools (Gazelle, buildozer, Code Search) to modify them correctly. Abstracted dependencies or dynamic generation makes tooling impossible.

**Example:**
```starlark
# BAD: Too much abstraction
COMMON_DEPS = [
    "//d:e",
    "//x/y:z",
]

cc_library(
    name = "a",
    srcs = ["a.cc"],
    deps = COMMON_DEPS + [...],
)
```

**Good: Explicit and readable**
```starlark
cc_library(
    name = "a",
    srcs = ["a.cc"],
    deps = [
        "//d:e",
        "//x/y:z",
        # ... specific deps for this target
    ],
)
```

## BUILD File Structure

**Recommended order (each element optional):**

1. **Package description** (comment explaining the package)
2. **load() statements** (import rules and macros)
3. **package() function** (set default settings)
4. **Rules and macro calls** (target declarations)

**Example:**
```starlark
# Test code for the Foo controller.

load("//testing:defs.bzl", "py_test")

package(default_testonly = True)

py_test(
    name = "foo_test",
    srcs = ["foo_test.py"],
    deps = ["//foo:lib"],
)
```

## Comments

- **Standalone comments** (section headers): Use blank line after
- **Attached comments** (rule-specific): Place directly above the rule

Distinction matters for automated changes (e.g., when deleting a rule).

---

See also: [[tools/buildifier]] for automatic formatting, [[patterns/target-naming]] for naming conventions
