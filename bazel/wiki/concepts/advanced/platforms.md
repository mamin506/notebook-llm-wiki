---
title: "Platforms & Toolchains (Recommended)"
category: "concepts"
level: "advanced"
status: "growing"
sources: ["Migrating to Platforms.md"]
tags: ["platforms", "toolchains", "recommended", "cross-compilation"]
graph-group: "concepts-advanced"
related: ["[[patterns/platform-configuration]]", "[[reference/platforms-legacy]]", "[[languages/cpp]]", "[[languages/java]]"]
last_updated: "2026-07-19"
---

# Platforms & Toolchains (Recommended)

The **Platforms and Toolchains** APIs are Bazel's modern, standardized approach for configuring multi-architecture and cross-compilation builds.

## What are Platforms?

A **platform** is a collection of machine properties (constraints):

```starlark
platform(
    name = "my_linux_arm64",
    constraint_values = [
        "@platforms//os:linux",
        "@platforms//cpu:aarch64",
    ],
)
```

This replaces legacy, ad-hoc approaches where each language used different flags:
- C++ used `--cpu`, `--crosstool_top`, `--compiler`
- Java used `--java_toolchain`, `--javabase`
- Android used `--android_cpu`, `--android_crosstool_top`

**Problem:** These flags didn't interoperate, making multi-language builds confusing.

## What are Toolchains?

A **toolchain** is a Starlark rule that declares language-specific build tools:

```starlark
cc_toolchain(
    name = "my_gcc",
    compiler = ":gcc_binary",  # Path to compiler executable
    ...
)
```

Toolchains declare:
- **Target compatibility** — What machines can be built for (`target_compatible_with = ["@platforms//os:linux"]`)
- **Execution compatibility** — What machines can run the tools (`exec_compatible_with = ["@platforms//os:macos"]`)

## Toolchain Resolution

When you run `bazel build //:app --platforms=//:my_linux_arm64`, Bazel **automatically selects** a toolchain that:
1. Can *run on* the build machine (exec host)
2. Can *target* the specified platform

Example:
```bash
$ bazel build //:app --platforms=//:my_linux_arm64
# Bazel selects a GCC toolchain that runs on macOS but targets Linux ARM64
```

## Constraint Settings & Values

**Constraint settings** define categories of machine properties:

```starlark
constraint_setting(name = "os")
constraint_setting(name = "cpu")
```

**Constraint values** are specific values within a setting:

```starlark
constraint_value(
    name = "linux",
    constraint_setting = ":os",
)
constraint_value(
    name = "aarch64",
    constraint_setting = ":cpu",
)
```

The common constraints are defined in `@platforms`:
- `@platforms//os:linux`, `@platforms//os:macos`, `@platforms//os:windows`
- `@platforms//cpu:x86_64`, `@platforms//cpu:aarch64`, `@platforms//cpu:arm`

## Language Support Status

### ✅ Fully Supported (Modern Bazel 7.0+)

**C++**
- Enabled by default in Bazel 7.0+
- Use: `bazel build //:app --platforms=//:myplatform`
- Replaces: `--cpu`, `--crosstool_top`, `--compiler`

**Java**
- Fully supported in modern Bazel
- Replaces: `--java_toolchain`, `--host_java_toolchain`, `--javabase`, `--host_javabase`

**Android**
- Enabled by default in Bazel 7.0+ (when `--incompatible_enable_android_toolchain_resolution` set)
- Use: `bazel build //:app --android_platforms=//:my_android_platform`
- Replaces: `--android_cpu`, `--android_crosstool_top`, `--fat_apk_cpu`

**Go**
- Rules from `bazelbuild/rules_go` fully support platforms

**Rust**
- Rules from `bazelbuild/rules_rust` fully support platforms

### ⚠️ Partial or Not Yet Supported

**Apple**
- Apple rules don't yet support platforms
- You can use platform APIs in mixed Apple + C++ builds with [platform mappings](#platform-mappings)

## Configuring Platforms

### Default Platform

When `--platforms` isn't set, Bazel defaults to `@platforms//host`, which represents your local build machine:

```bash
$ bazel build //:app  # Uses @platforms//host (your local machine)
```

### Custom Platforms

Define custom platforms in your project:

```starlark
# In your BUILD or MODULE.bazel
platform(
    name = "linux_arm64",
    constraint_values = [
        "@platforms//os:linux",
        "@platforms//cpu:aarch64",
    ],
)

# In a separate //platforms:BUILD
platform(
    name = "macos_x86",
    constraint_values = [
        "@platforms//os:macos",
        "@platforms//cpu:x86_64",
    ],
)
```

Use it:
```bash
$ bazel build //:app --platforms=//:linux_arm64
$ bazel build //:app --platforms=//platforms:macos_x86
```

### Registering Toolchains

Toolchains must be registered in `MODULE.bazel`:

```starlark
module(name = "my_project")

register_toolchains(
    "//toolchains:my_cc_toolchain",
    "//toolchains:my_java_toolchain",
)
```

Or at the command line:
```bash
$ bazel build //:app --extra_toolchains=//toolchains:my_cc_toolchain
```

## select() with Platforms

You can use `select()` to choose between alternatives based on platform constraints:

```starlark
cc_library(
    name = "crypto",
    srcs = select({
        "@platforms//os:linux": ["crypto_linux.cc"],
        "@platforms//os:macos": ["crypto_macos.cc"],
    }),
)
```

Instead of the old style:
```starlark
# Old (legacy)
cc_library(
    name = "crypto",
    srcs = select({
        "is_linux": ["crypto_linux.cc"],
        "is_macos": ["crypto_macos.cc"],
    }),
    ...
)

config_setting(
    name = "is_linux",
    values = {"cpu": "linux"},  # Depends on --cpu flag
)
```

## Platform Mappings

**Platform mappings** bridge legacy flag-based builds and modern platform-based builds during migration.

A `platform_mappings` file in your workspace root maps between the two:

```
platforms:
  # Map a platform to legacy flags
  //:my_android_platform
    --android_cpu=armeabi-v7a
    --fat_apk_cpu=armeabi-v7a

flags:
  # Map legacy flags to a platform
  --cpu=aarch64
    //:my_linux_arm64
```

This lets you gradually migrate while supporting both old and new build configurations.

## Benefits over Legacy Flags

| Aspect | Legacy Flags | Platforms |
|--------|--------------|-----------|
| **Interoperability** | ❌ Each language has own flags | ✅ Universal approach |
| **Clarity** | ❌ `--cpu=aarch64 --crosstool_top=...` | ✅ `--platforms=//:linux_arm64` |
| **Multi-language** | ❌ Confusing with mixed languages | ✅ Seamless cross-language support |
| **Debugging** | ❌ Flags scattered everywhere | ✅ Platform definition in one place |
| **Documentation** | ❌ Language-specific docs | ✅ Unified documentation |

## Migration Path

**For new projects:** Use platforms from day one. They're the modern, recommended approach.

**For existing projects:** Gradually migrate:
1. Define platforms for your build targets
2. Convert `config_setting` rules to use `constraint_values`
3. Update rules and transitions to use the toolchain API
4. Use platform mappings to support both old and new during transition
5. Once fully migrated, remove legacy flags and mappings

## Best Practices

1. **Define platforms explicitly** — Don't rely only on `@platforms//host`
2. **Use common constraints** — Declare OS and CPU in `@platforms`; keep custom constraints in your repo
3. **Register all toolchains** — Make sure every toolchain your build needs is registered
4. **Test cross-compilation** — Verify builds work with multiple `--platforms` values
5. **Document custom constraints** — If you define custom constraint settings, document what they mean
6. **Plan migration** — If moving from flags to platforms, use platform mappings during transition

---

**Key Insight:** Platforms are the future of Bazel configuration. They replace scattered flags with a unified, language-agnostic approach to multi-architecture builds.
