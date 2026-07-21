---
title: "Buildifier: BUILD File Formatter & Linter"
category: "tools"
level: "fundamentals"
status: "growing"
sources: ["BUILD Style Guide.md"]
tags: ["tools", "formatting", "linting", "automation"]
graph-group: "tools"
related: ["[[reference/build-conventions]]", "[[patterns/build-file-style]]"]
last_updated: "2026-07-19"
---

# Buildifier: BUILD File Formatter & Linter

## What is Buildifier?

**Buildifier** is the standard tool for:
- **Formatting** BUILD files into standard style
- **Linting** BUILD files for common issues
- **Code generation** and manipulation

It serves a similar role to `gofmt` in the Go ecosystem.

**Official Repository:** [bazelbuild/buildtools](https://github.com/bazelbuild/buildtools)

## Why Use Buildifier?

1. **Consistency** — All BUILD files formatted identically across your codebase
2. **No review debates** — Formatting is automated, not debated in code reviews
3. **Tooling support** — Enables other tools (buildozer, Gazelle) to work reliably
4. **Predictability** — Tools can read and modify BUILD files confidently

## Basic Usage

### Format a BUILD File
```bash
buildifier BUILD
```

### Format Multiple Files
```bash
buildifier -r path/to/project  # Recursive
```

### Check Without Modifying
```bash
buildifier --lint=off --mode=check BUILD
```

### Auto-Fix Linting Issues
```bash
buildifier --lint=fix BUILD
```

## Formatting Rules

Buildifier enforces the conventions documented in [[reference/build-conventions]]:

- **String literals** — Fuses concatenated strings (especially for labels)
- **Spacing** — Consistent spacing around `=` and operators
- **Indentation** — Uniform indentation (4 spaces)
- **Line breaks** — Readable multi-line formatting for long attribute lists
- **Comments** — Preserves and aligns comments appropriately

## Linting

Buildifier can detect common issues:

- **Unused load statements** — Imports that aren't used
- **Unreachable statements** — Code after return/break
- **Deprecated attributes** — Flags that are no longer supported
- **Naming conventions** — Violations of naming standards
- **And more** — See `buildifier --help` for full list

## Integration

### Pre-commit Hook
Add to `.git/hooks/pre-commit`:
```bash
#!/bin/bash
buildifier --mode=check $(git ls-files | grep BUILD)
```

### CI/CD Pipeline
```bash
buildifier --lint=fix --mode=check $(find . -name BUILD)
```

### IDE Integration
Most IDEs with Bazel support can run Buildifier on save.

## Common Formatting Output

**Input (Inconsistent):**
```starlark
cc_library(name="foo",srcs=["foo.cc"],deps=["//bar"])
```

**Output (Buildifier formatted):**
```starlark
cc_library(
    name = "foo",
    srcs = ["foo.cc"],
    deps = ["//bar"],
)
```

## Related Tools

- **buildozer** — Modify BUILD files programmatically (uses buildifier for formatting)
- **Gazelle** — Auto-generate BUILD files (uses buildifier for output)

---

See also: [[reference/build-conventions]], [[patterns/build-file-style]]
