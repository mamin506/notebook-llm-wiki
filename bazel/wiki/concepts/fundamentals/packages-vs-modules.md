---
title: "Packages vs Modules - Clear Comparison"
category: "concepts"
level: "fundamentals"
status: "seedling"
sources: []
tags: ["package", "module", "organization", "dependencies"]
related: ["[[concepts/fundamentals/packages]]", "[[concepts/fundamentals/module.bazel]]", "[[reference/modules-version-selection]]", "[[reference/external-dependencies]]"]
last_updated: "2026-07-20"
graph-group: "concepts-fundamentals"
---

# Packages vs Modules: Clear Comparison

These two organizational concepts work at **different layers** of Bazel. Understanding the distinction is crucial for working with Bazel effectively.

## Quick Comparison

| Aspect | Package | Module |
|--------|---------|--------|
| **Definition** | Directory with `BUILD.bazel` file | Project with `MODULE.bazel` file |
| **Layer** | Directory level | Project level |
| **Purpose** | Organize local source code | Manage external dependencies & versioning |
| **File** | `BUILD.bazel` | `MODULE.bazel` |
| **Scope** | Within a single workspace | Across workspaces / registries |
| **Number per Project** | **Multiple** (many directories) | **One** (at project root) |
| **Example** | `//src/app`, `//src/lib` | `my-project`, `rules_cc`, `rules_python` |
| **Visibility** | Local targets within workspace | Global (can be in Bazel Central Registry) |

## Packages: Local Code Organization

### What is a Package?

A **package** is a directory containing a `BUILD.bazel` file. It's the **smallest organizational unit** in Bazel.

```
src/my/project/BUILD.bazel      ← Package: //src/my/project
src/my/project/main.cc
src/my/project/main.h
src/my/project/tests/BUILD.bazel ← Package: //src/my/project/tests (separate)
src/my/project/tests/main_test.cc
```

### Purpose

- Group related source files
- Declare build targets (`cc_library`, `py_binary`, etc.)
- Define local dependencies between packages
- Control visibility and API surfaces

### Example: Multiple Packages in One Project

```
my-project/
├── MODULE.bazel
├── src/
│   ├── app/BUILD.bazel          ← Package: //src/app
│   │   ├── main.cc
│   │   └── BUILD.bazel
│   ├── lib/BUILD.bazel          ← Package: //src/lib
│   │   ├── lib.h
│   │   └── lib.cc
│   └── util/BUILD.bazel         ← Package: //src/util
│       ├── util.h
│       └── util.cc
└── tests/
    └── integration/BUILD.bazel  ← Package: //tests/integration
        └── integration_test.cc
```

This project has **4 packages** but only **1 module**.

### Package-Level Dependencies

Packages depend on other packages **locally**:

```starlark
# In src/app/BUILD.bazel
cc_binary(
    name = "myapp",
    srcs = ["main.cc"],
    deps = [
        "//src/lib:mylib",    # Depends on package //src/lib
        "//src/util:utils",   # Depends on package //src/util
    ],
)
```

## Modules: Project-Level Dependency Management

### What is a Module?

A **module** is a versioned Bazel project declared by a `MODULE.bazel` file at the **project root**. It's the unit of **external dependency management**.

```starlark
# MODULE.bazel (root of project)
module(
    name = "my-project",           # Module name
    version = "1.0.0",             # Module version
)

# Declare external dependencies
bazel_dep(name = "rules_cc", version = "0.1.1")
bazel_dep(name = "rules_python", version = "0.20.0")
bazel_dep(name = "googletest", version = "1.14.0")
```

### Purpose

- Declare your project as a **versioned, shareable module**
- Manage **external dependencies** from Bazel Central Registry
- Enable deterministic version resolution (MVS algorithm)
- Integrate with package managers (Maven, PyPI, npm)

### One Module, Multiple Packages

Every Bazel project is **one module** but contains **multiple packages**:

```
my-project/
├── MODULE.bazel              ← This project is MODULE "my-project" v1.0.0
├── src/
│   ├── app/BUILD.bazel       ← PACKAGE //src/app
│   ├── lib/BUILD.bazel       ← PACKAGE //src/lib
│   └── util/BUILD.bazel      ← PACKAGE //src/util
└── tests/
    └── integration/BUILD.bazel ← PACKAGE //tests/integration
```

### Module-Level Dependencies

Modules depend on **other modules** (external projects):

```starlark
# In MODULE.bazel
bazel_dep(name = "rules_python", version = "0.20.0")  # External module

# In src/app/BUILD.bazel
py_binary(
    name = "script",
    deps = [
        "//src/lib:mylib",              # Local package
        "@rules_python//python:lib",    # External module
    ],
)
```

The `@` prefix indicates an external module.

## Concrete Example: Putting It Together

```
my-calculator/                      ← A Bazel PROJECT
├── MODULE.bazel                    ← Declares this as MODULE "my-calculator" v2.0.0
├── src/
│   ├── calculator/
│   │   ├── BUILD.bazel             ← PACKAGE //src/calculator
│   │   ├── add.cc
│   │   ├── subtract.cc
│   │   └── calc.h
│   ├── ui/
│   │   ├── BUILD.bazel             ← PACKAGE //src/ui
│   │   ├── ui.cc
│   │   └── ui.h
│   └── BUILD.bazel                 ← PACKAGE //src
├── tests/
│   └── BUILD.bazel                 ← PACKAGE //tests
└── .bazelrc
```

### Dependency Structure

```starlark
# MODULE.bazel
module(name = "my-calculator", version = "2.0.0")
bazel_dep(name = "rules_cc", version = "0.1.1")  # External module
bazel_dep(name = "googletest", version = "1.14.0") # External module

# src/ui/BUILD.bazel
cc_library(
    name = "ui",
    srcs = ["ui.cc"],
    hdrs = ["ui.h"],
    deps = [
        "//src/calculator:calc",  # Depends on LOCAL package
    ],
)

# src/calculator/BUILD.bazel
cc_library(
    name = "calc",
    srcs = ["add.cc", "subtract.cc"],
    hdrs = ["calc.h"],
    deps = [
        "@rules_cc//cc:cc_library",  # Depends on EXTERNAL module
    ],
)

# tests/BUILD.bazel
cc_test(
    name = "calculator_test",
    srcs = ["calculator_test.cc"],
    deps = [
        "//src/calculator:calc",     # Local package
        "@googletest//:gtest_main",  # External module
    ],
)
```

## Common Confusion Points

### 1. "Is a package a module?"

**No.** A package is a directory; a module is an entire project. One module contains many packages.

### 2. "I have multiple BUILD.bazel files, so I have multiple modules?"

**No.** You still have **one module** (one MODULE.bazel). Multiple BUILD.bazel files means multiple packages.

### 3. "Can I have a MODULE.bazel inside a package?"

**No.** MODULE.bazel is only at the workspace root. You can't nest modules.

### 4. "Do I need both BUILD.bazel and MODULE.bazel?"

**Yes.** They serve different purposes:
- `MODULE.bazel` (root) — Declares your project and external dependencies
- `BUILD.bazel` (each directory) — Declares local targets and packages

## Decision Tree

```
I want to organize my local code
    └→ Use PACKAGE (BUILD.bazel in each directory)

I want to declare external dependencies (rules_cc, rules_python, etc.)
    └→ Use MODULE (MODULE.bazel at root)

I want to reference external code in my build rules
    └→ Use MODULE dependency with @ prefix (@rules_cc//:lib)

I want to reference another part of my project
    └→ Use PACKAGE dependency with // prefix (//src/lib:mylib)
```

## Best Practices

### Package Level
- ✅ One BUILD.bazel per directory with related code
- ✅ Clear package naming reflecting purpose
- ✅ Avoid deep nesting
- ✅ Use local dependencies first

### Module Level
- ✅ One MODULE.bazel at project root only
- ✅ List only direct dependencies
- ✅ Use specific versions initially (add constraints later)
- ✅ Commit MODULE.bazel.lock to git for reproducibility
- ✅ Keep MODULE.bazel simple and readable

## See Also

- [[concepts/fundamentals/packages]] — Package details and structure
- [[concepts/fundamentals/module.bazel]] — MODULE.bazel file format
- [[reference/modules-version-selection]] — Version resolution (MVS algorithm)
- [[reference/external-dependencies]] — External dependency management
- [[concepts/fundamentals/targets]] — What gets declared in BUILD.bazel
- [[concepts/fundamentals/labels]] — How to reference packages and modules
