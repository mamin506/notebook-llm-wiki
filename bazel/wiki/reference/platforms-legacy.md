---
title: "CPU Flags & Legacy Configuration (Deprecated)"
category: "reference"
level: "intermediate"
status: "stable"
sources: ["Migrating to Platforms.md"]
tags: ["legacy", "historical", "cpu-flags", "deprecated"]
graph-group: "reference"
related: ["[[concepts/advanced/platforms]]"]
last_updated: "2026-07-19"
---

# CPU Flags & Legacy Configuration (Deprecated)

The traditional approach to multi-platform builds used language-specific flags like `--cpu`, `--crosstool_top`, and `--compiler`.

## Current Status

- ✅ **Fully supported** — Existing builds using these flags continue to work
- ⚠️ **Deprecated** — Replaced by [[concepts/advanced/platforms]] in Bazel 7.0+
- ❌ **Will be removed** — Bazel will eventually deprecate and remove these flags
- ➡️ **Migrate to:** [[concepts/advanced/platforms]]

## Why Legacy Flags Were Problematic

**Fragmentation:** Each language had its own configuration approach:
- C++: `--cpu`, `--crosstool_top`, `--compiler`
- Java: `--java_toolchain`, `--javabase`, `--host_java_toolchain`, `--host_javabase`
- Android: `--android_cpu`, `--android_crosstool_top`, `--fat_apk_cpu`

**Consequences:**
- No interoperability between languages
- Multi-language builds were confusing and error-prone
- Inconsistent semantics across languages

## Legacy Build Command

```bash
# Old way (legacy)
bazel build //:my_cpp_project --cpu=aarch64 --crosstool_top=//toolchains:gcc

# New way (recommended)
bazel build //:my_cpp_project --platforms=//platforms:linux_arm64
```

## Modern Approach

[[concepts/advanced/platforms]] provides a unified, language-agnostic way to specify target architectures:

```bash
# Modern, recommended
bazel build //:my_cpp_project --platforms=//platforms:linux_arm64
```

## Migration Steps

If you're using legacy flags, migrate to platforms:

1. **Define platforms** — Create `//platforms:BUILD` with your target configurations
2. **Update rules** — Ensure your rules support the platforms API
3. **Convert select() statements** — Replace flag-based conditions with constraint-based conditions
4. **Use platform mappings** — During transition, use `platform_mappings` to support both old and new
5. **Remove legacy flags** — Once fully migrated, remove `--cpu`, `--crosstool_top`, etc.

See [[patterns/platform-configuration]] for detailed migration guidance.

---

**Recommended:** Use [[concepts/advanced/platforms]] for all new projects. Migrate existing projects to platforms at your earliest convenience.
