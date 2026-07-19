---
title: "Target Naming Conventions"
category: "patterns"
level: "fundamentals"
status: "growing"
sources: ["BUILD Style Guide.md"]
tags: ["naming", "conventions", "targets"]
related: ["[[patterns/build-file-style]]", "[[concepts/fundamentals/targets]]"]
last_updated: "2026-07-19"
---

# Target Naming Conventions

## Basic Principles

- **Descriptive names** are better than short names
- If a target contains one source file, derive the name from that file
  - Example: `cc_library` for `chat.cc` → name `"chat"`
  - Example: `java_library` for `DirectMessage.java` → name `"direct_message"`
- **Eponymous targets** (same name as directory) should provide the main functionality
- Prefer short names for eponymous targets: `//x` instead of `//x:x`

## Naming Conventions by Language

### General (snake_case)
Use lowercase with underscores for most targets.

### Java
- **Libraries**: snake_case (e.g., `"direct_message"`)
- **Binaries & Tests**: Upper CamelCase (e.g., `"MyBinary"`, `"MyTest"`)
  - Allows binary name to match a source file
  - For `java_test`, enables automatic `test_class` inference

### C++
Use snake_case.

### Proto
- `proto_library`: Name ends with `_proto`
  - Example: `"message_proto"`
- Language-specific libraries:
  - `cc_proto_library`: ends with `_cc_proto`
  - `java_proto_library`: ends with `_java_proto`
  - `java_lite_proto_library`: ends with `_java_proto_lite`

## Variants & Suffixes

Use suffixes to disambiguate variants:
- `:foo_dev`, `:foo_prod` (environment variants)
- `:bar_x86`, `:bar_x64` (architecture variants)
- `:lib_test`, `:lib_unittest`, `:LibTest` (test targets)

## What to Avoid

- **Reserved names**: `all`, `__pkg__`, `__subpackages__` (special semantic meaning)
- **Meaningless suffixes**: `_lib`, `_library` (unless disambiguating a binary/library pair)
- **Acronyms** without context

## References

- Relative references (same package): `:target_name`
- Absolute references (other packages): `//package:target`
- Source files: No prefix (`:` is only for generated/rules)
  - Example: `srcs = ["foo.cc"]` ✓
  - Example: `srcs = [":foo.cc"]` ✗

---

See also: [[patterns/build-file-style]]
