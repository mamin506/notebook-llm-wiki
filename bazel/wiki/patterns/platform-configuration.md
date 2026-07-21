---
title: "Configuring Platforms for Your Project"
category: "patterns"
level: "intermediate"
status: "growing"
sources: ["Migrating to Platforms.md"]
tags: ["platforms", "configuration", "cross-compilation", "best-practices"]
graph-group: "patterns"
related: ["[[concepts/advanced/platforms]]", "[[concepts/advanced/hermeticity]]", "[[languages/cpp]]", "[[languages/java]]"]
last_updated: "2026-07-19"
---

# Configuring Platforms for Your Project

A practical guide to setting up platform-based builds in your Bazel project.

## Step 1: Define Your Target Platforms

Create a `platforms/BUILD` file in your project root (or wherever you keep platform definitions):

```starlark
# //platforms:BUILD

platform(
    name = "linux_x86_64",
    constraint_values = [
        "@platforms//os:linux",
        "@platforms//cpu:x86_64",
    ],
)

platform(
    name = "linux_arm64",
    constraint_values = [
        "@platforms//os:linux",
        "@platforms//cpu:aarch64",
    ],
)

platform(
    name = "macos_x86_64",
    constraint_values = [
        "@platforms//os:macos",
        "@platforms//cpu:x86_64",
    ],
)

platform(
    name = "macos_arm64",
    constraint_values = [
        "@platforms//os:macos",
        "@platforms//cpu:aarch64",
    ],
)

platform(
    name = "windows_x86_64",
    constraint_values = [
        "@platforms//os:windows",
        "@platforms//cpu:x86_64",
    ],
)
```

## Step 2: Register Your Toolchains

In `MODULE.bazel`, register all toolchains your project uses:

```starlark
# MODULE.bazel
module(name = "my_project", version = "1.0.0")

# C++ Toolchain
register_toolchains(
    "//toolchains/cc:gcc_toolchain",
    "//toolchains/cc:clang_toolchain",
)

# Java Toolchain
register_toolchains(
    "//toolchains/java:java11_toolchain",
    "//toolchains/java:java17_toolchain",
)
```

## Step 3: Configure select() Rules

Use `select()` with constraint values instead of legacy flags:

```starlark
# //myapp:BUILD

cc_binary(
    name = "myapp",
    srcs = ["main.cc"],
    deps = [":crypto"],
)

cc_library(
    name = "crypto",
    srcs = ["crypto_generic.cc"],
    srcs_select = {
        "@platforms//os:linux": ["crypto_linux.cc"],
        "@platforms//os:macos": ["crypto_macos.cc"],
        "@platforms//os:windows": ["crypto_windows.cc"],
    },
)

# Or using config_setting for more complex conditions:
config_setting(
    name = "is_arm64",
    constraint_values = [
        "@platforms//cpu:aarch64",
    ],
)

cc_library(
    name = "simd_lib",
    srcs = select({
        ":is_arm64": ["simd_arm64.cc"],
        "@platforms//cpu:x86_64": ["simd_x86_64.cc"],
        "//conditions:default": ["simd_generic.cc"],
    }),
)
```

## Step 4: Use Platforms in Builds

Build for different platforms:

```bash
# Local machine (uses default @platforms//host)
$ bazel build //:myapp

# Linux x86_64
$ bazel build //:myapp --platforms=//platforms:linux_x86_64

# macOS ARM64 (Apple Silicon)
$ bazel build //:myapp --platforms=//platforms:macos_arm64

# Cross-compile for Android (if using Android rules)
$ bazel build //:myapp --platforms=//platforms:android_arm64
```

## Step 5: Handle Multi-Language Builds

If your project mixes languages (C++ + Java, etc.), ensure each language's toolchain is registered:

```starlark
# In MODULE.bazel or workspace root

# C++ toolchain
register_toolchains("//toolchains:cc_toolchain")

# Java toolchain
register_toolchains("//toolchains:java_toolchain")

# Python (if you have custom Python toolchain)
register_toolchains("//toolchains:python_toolchain")
```

## Step 6: Gradual Migration (Legacy Fallback)

If you're migrating from legacy flags, use platform mappings to support both simultaneously:

```
# platform_mappings file in workspace root

platforms:
  # Map platform to legacy C++ flags
  //platforms:linux_arm64
    --cpu=aarch64
    --crosstool_top=//toolchains/cc:gcc_toolchain

flags:
  # Map legacy flags to platform
  --cpu=aarch64
    //platforms:linux_arm64

  --cpu=x86_64
    //platforms:linux_x86_64
```

This allows builds with both:
```bash
# New style (recommended)
$ bazel build //:app --platforms=//platforms:linux_arm64

# Old style (still works during migration)
$ bazel build //:app --cpu=aarch64
```

## Common Issues & Solutions

### Issue: Toolchain Not Found

**Error:** `No matching toolchain for (platform|CPU)`

**Causes:**
- Toolchain not registered in `MODULE.bazel`
- Toolchain's `target_compatible_with` doesn't match platform

**Solution:**
```starlark
# Check toolchain definition
cc_toolchain(
    name = "my_toolchain",
    target_compatible_with = [
        "@platforms//os:linux",
        "@platforms//cpu:x86_64",
    ],
)

# Ensure it's registered
# In MODULE.bazel:
register_toolchains("//toolchains:my_toolchain")
```

### Issue: select() Condition Not Matching

**Error:** Build succeeds but uses wrong `select()` branch

**Causes:**
- Constraint value in `config_setting` doesn't match platform
- Typo in constraint reference

**Solution:**
```starlark
# Correct way
config_setting(
    name = "is_linux",
    constraint_values = [
        "@platforms//os:linux",  # Exact constraint path
    ],
)

# Use in select()
srcs = select({
    ":is_linux": ["linux.cc"],
    "//conditions:default": ["generic.cc"],
})
```

### Issue: Cross-Compilation Fails

**Error:** Build selects host toolchain instead of cross toolchain

**Causes:**
- Toolchain's `exec_compatible_with` doesn't match build machine
- Toolchain not registered

**Solution:**
```starlark
# Toolchain that runs on macOS but targets Linux
cc_toolchain(
    name = "macos_to_linux",
    # Tools can run on macOS
    exec_compatible_with = [
        "@platforms//os:macos",
        "@platforms//cpu:x86_64",
    ],
    # Tools can build for Linux
    target_compatible_with = [
        "@platforms//os:linux",
        "@platforms//cpu:x86_64",
    ],
)
```

## Testing Your Platform Configuration

### Verify Local Builds Work

```bash
# Should build successfully
$ bazel build //... --platforms=@platforms//host
```

### Test Cross-Compilation

```bash
# Verify cross-compilation (Linux from macOS, for example)
$ bazel build //... --platforms=//platforms:linux_x86_64
$ bazel build //... --platforms=//platforms:macos_arm64
```

### Check Toolchain Selection

```bash
# Verbose output shows which toolchain was selected
$ bazel build //:myapp -s --platforms=//platforms:linux_arm64 2>&1 | grep toolchain
```

## Best Practices

1. **Define platforms once** — Store all platform definitions in a shared location (e.g., `//platforms:BUILD`)
2. **Use common constraints** — Leverage `@platforms//os` and `@platforms//cpu`; avoid custom constraints if possible
3. **Document custom platforms** — If you define custom constraints, document their semantics
4. **Separate toolchain definitions** — Keep toolchain definitions separate from platform definitions
5. **Test all platforms** — CI should verify builds succeed on all target platforms
6. **Migrate gradually** — If switching from flags to platforms, use platform mappings during transition

## Example: Complete Platform Setup

```starlark
# //platforms:BUILD
platform(
    name = "linux_x86_64",
    constraint_values = [
        "@platforms//os:linux",
        "@platforms//cpu:x86_64",
    ],
)

# //toolchains/cc:BUILD
cc_toolchain(
    name = "gcc_linux_x86_64",
    compiler = "@gcc//:bin/gcc",
    target_compatible_with = [
        "@platforms//os:linux",
        "@platforms//cpu:x86_64",
    ],
    exec_compatible_with = [
        "@platforms//os:linux",
        "@platforms//cpu:x86_64",
    ],
)

# //MODULE.bazel
module(name = "my_project")

register_toolchains(
    "//toolchains/cc:gcc_linux_x86_64",
)

# Usage
# $ bazel build //:app --platforms=//platforms:linux_x86_64
```

---

**Next Steps:** Once platforms are configured, see [[concepts/advanced/platforms]] for deeper understanding, and language-specific guides like [[languages/cpp]] and [[languages/java]] for language-specific setup.
