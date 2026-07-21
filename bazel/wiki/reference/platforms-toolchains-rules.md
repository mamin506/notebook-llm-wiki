---
title: "Platforms and Toolchains Rules Reference"
category: "reference"
level: "advanced"
status: "seedling"
sources: ["Platforms and Toolchains Rules.md"]
tags: ["#platforms", "#toolchains", "#constraints", "#configuration"]
related: ["[[concepts/advanced/platforms]]", "[[reference/build-options]]", "[[patterns/platform-configuration]]"]
last_updated: "2026-07-19"
graph-group: "reference"
---

# Platforms and Toolchains Rules Reference

Rules for modeling hardware platforms, defining constraints, and configuring toolchains for cross-platform builds.

## Overview

The platform and toolchain system allows Bazel to:
- Define execution and target platforms with specific constraints
- Select appropriate toolchains based on platform properties
- Cross-compile for different architectures and operating systems
- Model custom hardware platforms and constraint dimensions

## Core Rules

### constraint_setting

Introduces a new constraint type that platforms can have values for.

**Attributes:**
- `default_constraint_value`: The default value for this constraint (optional)
- `refines_constraint_value`: A constraint value this setting refines (optional)

**Example:**
```python
# Define a new constraint type
constraint_setting(
    name = "glibc_version",
)

# Define values for the constraint
constraint_value(
    name = "glibc_2_25",
    constraint_setting = ":glibc_version",
)

constraint_value(
    name = "glibc_2_30",
    constraint_setting = ":glibc_version",
)

# Use in platform:
platform(
    name = "linux_with_glibc_2_25",
    constraint_values = [
        "@platforms//os:linux",
        "@platforms//cpu:x86_64",
        ":glibc_2_25",
    ],
)
```

**Use Cases:**
- Define custom hardware features (CPU type, compiler version, etc.)
- Model OS variants or library versions
- Capture build environment properties

### constraint_value

Defines a specific value for a constraint setting.

**Attributes:**
- `constraint_setting`: The setting this value belongs to (required)

**Example:**
```python
constraint_value(
    name = "arm64",
    constraint_setting = "@platforms//cpu:cpu",
)
```

**Predefined Constraint Values:**
Bazel includes standard constraints via `@platforms`:
- `@platforms//os:linux`, `@platforms//os:macos`, `@platforms//os:windows`
- `@platforms//cpu:x86_64`, `@platforms//cpu:arm64`, `@platforms//cpu:arm`
- `@platforms//os:windows` with `@platforms//cpu:x86_64` (Windows with x86)

### platform

Combines constraint values to describe a complete platform.

**Attributes:**
- `constraint_values`: List of constraint values this platform includes
- `exec_properties`: Platform-specific execution properties (for remote execution)

**Example:**
```python
platform(
    name = "linux_x86_64",
    constraint_values = [
        "@platforms//os:linux",
        "@platforms//cpu:x86_64",
    ],
)

platform(
    name = "macos_arm64",
    constraint_values = [
        "@platforms//os:macos",
        "@platforms//cpu:arm64",
    ],
)
```

### toolchain

Registers a toolchain for a specific platform.

**Attributes:**
- `toolchain_type`: The type of toolchain (e.g., `@bazel_tools//tools/cpp:toolchain_type`)
- `toolchain`: The actual toolchain implementation
- `target_compatible_with`: Constraints this toolchain applies to
- `exec_compatible_with`: Constraints the execution platform must satisfy

**Example:**
```python
toolchain(
    name = "cpp_clang",
    toolchain_type = "@bazel_tools//tools/cpp:toolchain_type",
    toolchain = ":cpp_clang_impl",
    target_compatible_with = [
        "@platforms//os:linux",
        "@platforms//cpu:x86_64",
    ],
)
```

## Constraint System

### Platform Definition

A platform is a set of constraint values that describe a build environment:

```python
constraint_setting(name = "compiler")
constraint_value(name = "gcc", constraint_setting = ":compiler")
constraint_value(name = "clang", constraint_setting = ":compiler")

platform(
    name = "clang_platform",
    constraint_values = [
        ":clang",
        "@platforms//os:linux",
    ],
)
```

### Refining Constraints

Use `refines_constraint_value` to create constraint hierarchies:

```python
constraint_setting(
    name = "os",
)

constraint_value(
    name = "linux",
    constraint_setting = ":os",
)

constraint_setting(
    name = "linux_variant",
    refines_constraint_value = ":linux",  # Only valid if OS is Linux
)

constraint_value(
    name = "ubuntu_18_04",
    constraint_setting = ":linux_variant",
)

# Platform can now specify both:
platform(
    name = "ubuntu_build",
    constraint_values = [
        ":linux",
        ":ubuntu_18_04",  # Automatically implies :linux
    ],
)
```

## Toolchain Selection

Bazel selects toolchains by matching platform constraints:

```python
# Register multiple toolchains
toolchain(
    name = "cpp_linux_clang",
    toolchain_type = "@bazel_tools//tools/cpp:toolchain_type",
    toolchain = ":clang_toolchain_impl",
    target_compatible_with = [
        "@platforms//os:linux",
    ],
)

toolchain(
    name = "cpp_macos_clang",
    toolchain_type = "@bazel_tools//tools/cpp:toolchain_type",
    toolchain = ":macos_clang_toolchain_impl",
    target_compatible_with = [
        "@platforms//os:macos",
    ],
)

# When building for Linux, :cpp_linux_clang is selected
# When building for macOS, :cpp_macos_clang is selected
```

## Common Patterns

### Custom Hardware Platform

```python
# Define custom hardware constraint
constraint_setting(name = "board")

constraint_value(
    name = "raspberry_pi",
    constraint_setting = ":board",
)

# Define platform for the board
platform(
    name = "rpi_build",
    constraint_values = [
        "@platforms//os:linux",
        "@platforms//cpu:arm",
        ":raspberry_pi",
    ],
)

# Use in build:
# bazel build //firmware:app --platforms=//platforms:rpi_build
```

### Multi-Target Build

```python
constraint_setting(name = "target_os")

constraint_value(name = "ios", constraint_setting = ":target_os")
constraint_value(name = "android", constraint_setting = ":target_os")
constraint_value(name = "linux", constraint_setting = ":target_os")

# Build for multiple targets
# bazel build //app:app \
#     --platforms=//platforms:ios \
#     --platforms=//platforms:android \
#     --platforms=//platforms:linux
```

### Compiler Selection

```python
constraint_setting(name = "compiler")

constraint_value(name = "gcc", constraint_setting = ":compiler")
constraint_value(name = "clang", constraint_setting = ":compiler")

toolchain(
    name = "gcc_toolchain",
    toolchain_type = "@bazel_tools//tools/cpp:toolchain_type",
    toolchain = ":gcc_impl",
    target_compatible_with = [":gcc"],
)

toolchain(
    name = "clang_toolchain",
    toolchain_type = "@bazel_tools//tools/cpp:toolchain_type",
    toolchain = ":clang_impl",
    target_compatible_with = [":clang"],
)

# Build with GCC:
# bazel build //app:app --extra_toolchains=//toolchains:gcc_toolchain
```

## Platform Matching

### config_setting

Use `constraint_values` in `config_setting` to match platforms:

```python
config_setting(
    name = "on_linux",
    constraint_values = [
        "@platforms//os:linux",
    ],
)

cc_library(
    name = "lib",
    srcs = ["lib.cc"] + select({
        ":on_linux": ["linux_specific.cc"],
        "@platforms//os:macos": ["macos_specific.cc"],
    }),
)
```

### Platform Constraints in BUILD

```python
# This target only builds on Linux
cc_binary(
    name = "linux_only_app",
    srcs = ["linux_app.cc"],
    target_compatible_with = [
        "@platforms//os:linux",
    ],
)

# Will fail to build on non-Linux platforms:
# bazel build //app:linux_only_app --platforms=//platforms:macos
# ERROR: Target //app:linux_only_app is not compatible with macOS
```

## Execution Properties

For remote execution, specify platform-specific properties:

```python
platform(
    name = "docker_rbe",
    constraint_values = [
        "@platforms//os:linux",
        "@platforms//cpu:x86_64",
    ],
    exec_properties = {
        "container-image": "docker://bazel/ubuntu:20.04",
        "cores": "8",
        "mem": "16g",
    },
)

# Build with RBE:
# bazel build //app:app --platforms=//platforms:docker_rbe \
#     --remote_executor=grpcs://rbe.example.com
```

## Best Practices

1. **Use Standard Constraints** — Use `@platforms` for common OS/CPU constraints
2. **Name Clearly** — Use descriptive names: `linux_x86_64_glibc_2_31`
3. **Document Constraints** — Add comments explaining custom constraints
4. **Avoid Over-Specificity** — Only constrain what actually matters
5. **Test Platform Matching** — Verify builds succeed/fail as expected on target platforms

## See Also

- [[concepts/advanced/platforms]] — Platform concepts and usage
- [[patterns/platform-configuration]] — Setting up platforms in projects
- [[reference/build-options]] — Build flags and platform options
- [[reference/cli-reference]] — `--platforms` and `--extra_toolchains` flags
