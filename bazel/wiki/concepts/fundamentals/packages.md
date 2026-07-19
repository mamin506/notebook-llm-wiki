---
title: "Packages: The Unit of Organization"
category: "concepts"
level: "fundamentals"
status: "growing"
sources: ["Repositories, workspaces, packages, and targets.md"]
tags: ["core", "package", "organization"]
related: ["[[concepts/fundamentals/targets]]", "[[concepts/fundamentals/build-files]]", "[[concepts/fundamentals/labels]]"]
last_updated: "2026-07-19"
---

# Packages: The Unit of Organization

A **package** is the fundamental unit of code organization in Bazel. It's a collection of related source files and a specification of what software can be built from them.

## Definition

A package is **a directory containing a `BUILD` file** (named either `BUILD` or `BUILD.bazel`).

A package includes:
- All files in its directory
- All subdirectories beneath it, **except those that contain their own `BUILD` file**

**Key principle:** No file or directory can belong to two different packages.

## Example

```
src/my/app/BUILD              ← Package: //my/app
src/my/app/app.cc
src/my/app/data/input.txt     ← Part of //my/app (no BUILD file here)
src/my/app/tests/BUILD        ← Package: //my/app/tests (separate package)
src/my/app/tests/test.cc
```

In this structure:
- `//my/app` is a package (has BUILD file)
- `//my/app/tests` is a separate package (has its own BUILD file)
- `my/app/data/` is NOT a package; it belongs to `//my/app`

## Package Organization

Packages are referenced by their path from the workspace root:

- `//src` — Package in `src/BUILD`
- `//src/lib` — Package in `src/lib/BUILD`
- `//` — Root package (in workspace root BUILD)

## Packages and Dependencies

Packages can have dependencies on other packages:

```starlark
# In //src/app/BUILD
cc_binary(
    name = "my_app",
    srcs = ["app.cc"],
    deps = [
        "//src/lib:mylib",  # Depends on //src/lib package
    ],
)
```

A package's outputs always belong to that package; you cannot generate files into another package.

## Best Practices

- **One BUILD file per directory** with related code
- **Clear package naming** that reflects the code's purpose
- **Avoid deep nesting** — keep packages at reasonable depth
- **Use package-local dependencies** first, then cross-package

---

See also: [[concepts/fundamentals/targets]], [[concepts/fundamentals/build-files]], [[concepts/fundamentals/labels]]
