---
title: "BUILD (Legacy Filename)"
category: "reference"
level: "fundamentals"
status: "stable"
sources: ["BUILD files.md"]
tags: ["legacy", "historical", "build-files"]
graph-group: "reference"
related: ["[[concepts/fundamentals/build.bazel]]"]
last_updated: "2026-07-19"
---

# BUILD (Legacy Filename)

**`BUILD`** is the traditional filename for Bazel build files. It remains fully supported but is **no longer recommended** for new projects.

## Current Status

- ✅ **Fully supported** — Bazel treats BUILD identically to BUILD.bazel
- ⚠️ **Legacy** — Used in existing projects, not recommended for new code
- ➡️ **Migrate to:** [[concepts/fundamentals/build.bazel]]

## Why Not BUILD?

The filename `BUILD` is ambiguous. Many build systems use this name (Maven, Gradle, etc.), making it unclear which system owns the file.

`BUILD.bazel` is explicit and unambiguous.

## Using BUILD Today

If you're working in an existing codebase using `BUILD`:
- Keep the existing filename for consistency
- Follow the same structure and principles as [[concepts/fundamentals/build.bazel]]
- Consider renaming to `BUILD.bazel` during refactoring

---

**Recommended:** Use [[concepts/fundamentals/build.bazel]] for all new files and projects.
