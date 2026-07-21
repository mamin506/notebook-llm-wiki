---
title: "General Rules Reference"
category: "reference"
level: "intermediate"
status: "growing"
sources: ["General Rules.md"]
tags: ["#rules", "#reference", "#utilities"]
related: ["[[concepts/fundamentals/targets]]", "[[reference/cli-reference]]", "[[patterns/dependency-management]]"]
last_updated: "2026-07-19"
graph-group: "reference"
---

# General Rules Reference

General-purpose rules that aren't tied to a specific language. These are building blocks for advanced build workflows and configuration.

## alias

Creates an alternate name for an existing target. Useful for renaming without changing all dependents.

**Attributes:**
- `actual`: The target this alias refers to (required)

**Example:**
```python
filegroup(
    name = "data",
    srcs = ["input.txt"],
)

alias(
    name = "public_data",
    actual = ":data",
)
```

**Use Cases:**
- Renaming a target without breaking all dependents
- Exposing internal targets via a cleaner public name
- Storing a `select()` call for reuse across multiple targets

**Limitations:**
- `test_suite` cannot be aliased
- Tests aren't run if their alias is mentioned on the command line (use `test_suite` for that)

## config_setting

Matches a configuration state (build flags, platform constraints) for use with `select()` expressions.

**Attributes:**
- `values`: Dictionary of build flag values to match (e.g., `{"compilation_mode": "opt"}`)
- `flag_values`: Dictionary of user-defined flag values (e.g., `{"//flags:feature": "1"}`)
- `constraint_values`: List of platform constraint values to match

**Example:**
```python
config_setting(
    name = "opt_mode",
    values = {"compilation_mode": "opt"},
)

cc_binary(
    name = "myapp",
    srcs = ["main.cc"],
    copts = select({
        ":opt_mode": ["-O3"],
        "//conditions:default": ["-O0"],
    }),
)
```

**Common Build Flags:**
- `compilation_mode`: `fastbuild`, `dbg`, `opt`
- `cpu`: Target CPU (e.g., `x86_64`, `arm64`)
- `os`: Target OS (e.g., `linux`, `macos`, `windows`)

**Use Cases:**
- Conditional compilation based on build mode
- Platform-specific code paths
- Feature flags and experiments

## filegroup

Groups files for use by other rules (data, sources, configuration, etc.).

**Attributes:**
- `srcs`: List of files or targets to include
- `data`: Files needed at runtime (rarely used; usually `srcs` is sufficient)

**Example:**
```python
filegroup(
    name = "test_data",
    srcs = glob(["testdata/**/*.txt"]),
)

cc_test(
    name = "parser_test",
    srcs = ["parser_test.cc"],
    data = [":test_data"],
)
```

**Use Cases:**
- Grouping multiple files for distribution
- Creating data sets for tests
- Declaring configurations (e.g., `filegroup` for config files)

## genrule

Executes an arbitrary command to generate build artifacts.

**Attributes:**
- `srcs`: Input files for the command
- `outs`: Output files the command produces
- `cmd`: Shell command to run (access inputs/outputs via `$(location ...)` macro)
- `tools`: Tools (binaries) the command needs
- `message`: Human-readable description of what the rule does

**Example:**
```python
genrule(
    name = "generate_code",
    srcs = ["template.txt"],
    outs = ["generated.cc"],
    cmd = "cat $(location :template.txt) > $@",  # $@ = output file
    message = "Generating C++ source",
)

cc_library(
    name = "generated_lib",
    srcs = [":generate_code"],
)
```

**Important:**
- `genrule` should be a **last resort** for custom builds
- Better to write a custom Starlark rule if possible (more testable, cacheable, hermeticity)
- Outputs must be deterministic (no timestamps, random data, etc.)

**Macros:**
- `$(location //path:target)`: Path to a file or target output
- `$@`: Output file path
- `$<`: First input file path

## test_suite

Groups multiple test targets for convenient execution.

**Attributes:**
- `tests`: List of test targets to include

**Example:**
```python
test_suite(
    name = "all_tests",
    tests = [
        ":unit_test",
        ":integration_test",
        "//subdir:tests",
    ],
)
```

**Command:**
```
bazel test //:all_tests  # Runs all grouped tests
```

**Use Cases:**
- Grouping related tests
- Defining test suites for different purposes (fast, slow, integration, etc.)
- Reducing the number of targets to type on the command line

## package

Module-level declaration (not a rule, but key for configuration). Sets defaults for all targets in a BUILD file.

**Attributes:**
- `default_visibility`: Default visibility for targets in this package
- `licenses`: License declaration for the package
- `default_test_only`: Mark all tests as test-only by default
- `features`: Features to enable/disable for all targets

**Example:**
```python
package(
    default_visibility = ["//visibility:public"],
    licenses = ["notice"],
    features = ["layering_check"],  # C++ header checking
)
```

## exports_files

Exports source files from a package (makes them available to other packages).

**Attributes:**
- `srcs`: Files to export
- `visibility`: Who can see these files

**Example:**
```python
exports_files([
    "README.md",
    "LICENSE",
])

exports_files(
    glob(["*.proto"]),
    visibility = ["//visibility:public"],
)
```

## load

Imports Starlark symbols (rules, functions) from other `.bzl` files.

**Example:**
```python
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")
load(":macros.bzl", "my_macro")
```

(More detail on Starlark in custom rules documentation.)

## See Also

- [[concepts/fundamentals/targets]] — Target concepts
- [[reference/cli-reference]] — Bazel CLI and commands
- [[patterns/dependency-management]] — Dependency management patterns
