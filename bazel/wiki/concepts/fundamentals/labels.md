---
title: "Labels: Identifying Targets"
category: "concepts"
level: "fundamentals"
status: "growing"
sources: ["Labels    Bazel.md"]
tags: ["core", "labels", "references"]
related: ["[[concepts/fundamentals/packages]]", "[[concepts/fundamentals/targets]]"]
last_updated: "2026-07-19"
---

# Labels: Target Identifiers

A **label** is a unique identifier for a target. It specifies both the target and how to reference it.

## Full Label Syntax

```
@@canonical_repo//package/path:target_name
```

- `@@canonical_repo` — Repository name (double @ means canonical)
- `//package/path` — Fully-qualified package path
- `:target_name` — Target name within that package

## Shorthand Forms

### Same Repository (Most Common)

Omit the repository part:

```
//package/path:target_name
```

### Same Package

Omit the package path (and optional colon):

```
:target_name
target_name    # Colon optional for files, conventional for rules
```

### Eponymous Targets

When target name matches package name, both can be omitted:

```
//my/app/lib     # Shorthand for //my/app/lib:lib
```

## Label Examples

```
@@rules_cc//cc:cc_library              # Full canonical form
@rules_cc//cc:cc_library               # Apparent form
//my/app:app_binary                    # Same repo
//my/app                               # Shorthand (same as //my/app:app)
:app_binary                            # Same package
app_binary                             # Same package (file shorthand)
//my/app:testdata/input.txt            # File in subdirectory
```

## Important Rules

- **Labels are strings** — Always use string literals, never concatenate
- **Relative paths not allowed** — Must include package path for cross-package references
- **No `..` references** — Cannot refer to parent package with `../`
- **No `/` in rule names** — Avoid slashes in rule target names (confusing)
- **Avoid metacharacters** — Use alphanumeric + `-_@`

## Common Mistake

```starlark
# WRONG: "testdata" is a different package
srcs = ["testdata/file.txt"]

# CORRECT: Use full path
srcs = ["//my/app/testdata:file.txt"]
```

---

See also: [[concepts/fundamentals/packages]], [[concepts/fundamentals/targets]]
