---
title: "Organizing Multiple Toolchains in Monorepos"
category: "patterns"
level: "intermediate"
status: "seedling"
sources: []
tags: ["monorepo", "toolchains", "architecture", "multi-language"]
related: ["[[concepts/advanced/module-extensions]]", "[[reference/platforms-toolchains-rules.md]]", "[[patterns/dependency-management]]", "[[patterns/platform-configuration]]"]
last_updated: "2026-07-20"
graph-group: "patterns"
---

# Organizing Multiple Toolchains in Monorepos

Managing multiple toolchains (Python, C/C++, embedded systems, etc.) in a single Bazel monorepo is a common architectural challenge. This guide covers strategies for clean, maintainable organization.

## The Problem

A large monorepo often contains projects in different languages:
- Python services and scripts
- C/C++ libraries and binaries
- Embedded C firmware
- Java applications
- TypeScript frontends
- etc.

Each language ecosystem has its own toolchain requirements:
- Python needs `rules_python` + Python interpreter + package manager (pip/poetry)
- C/C++ needs `rules_cc` + compiler (GCC, Clang, MSVC)
- Embedded C needs specialized ARM/cross-compilation toolchains
- Java needs `rules_jvm` + Java compiler + Maven/Gradle integration

**The naive approach—putting everything in one MODULE.bazel—leads to:**
- Bloated configuration (unrelated toolchains mixed together)
- All projects paying the cost of all toolchain dependencies
- Difficult to reason about which projects need what
- Hard to upgrade or swap toolchains for specific projects

## Solution: Three Strategies

### Strategy 1: Module Extensions (Recommended)

**Best for:** Large, organized monorepos with clear project separation

Use module extensions to group toolchains by language/type.

```starlark
# MODULE.bazel (root, minimal and clean)
module(name = "my-monorepo", version = "1.0.0")

# Declare dependencies (no direct toolchain config here)
bazel_dep(name = "rules_python", version = "0.20.0")
bazel_dep(name = "rules_cc", version = "0.1.1")
bazel_dep(name = "embedded_tools", version = "1.0.0")
bazel_dep(name = "rules_jvm", version = "5.0.0")

# Use extensions to organize toolchains by language
python = use_extension("@rules_python//python:extensions.bzl", "python")
python.toolchain(python_version = "3.11")
python.toolchain(python_version = "3.10", is_default = False)
use_repo(python, "python_3_11", "python_3_10")

cc = use_extension("@rules_cc//cc:extensions.bzl", "cc")
cc.configure(
    gcc_version = "11.0",
    clang_version = "14.0",
)
use_repo(cc, "gcc_11", "clang_14")

# Custom extension for embedded toolchain
embedded = use_extension("@embedded_tools//:extensions.bzl", "embedded")
embedded.arm_toolchain(vendor = "arm", version = "10.3-2021.10")
embedded.risc_v_toolchain(vendor = "sifive", version = "2021.08")
use_repo(embedded, "arm_none_eabi", "riscv64_gnu")

# Java ecosystem
maven = use_extension("@rules_jvm//java:extensions.bzl", "maven")
maven.install(
    artifacts = [
        "org.junit.jupiter:junit-jupiter-api:5.9.0",
        "com.google.guava:guava:31.1-jre",
    ],
)
use_repo(maven, "maven")
```

**Organization in BUILD files:**

```starlark
# python_services/auth_service/BUILD.bazel
py_binary(
    name = "auth_service",
    srcs = ["main.py"],
    deps = [
        "//python_services/lib:auth_lib",
        "@python_3_11//:lib",  # Explicit Python version
    ],
)

# cpp_libraries/core/BUILD.bazel
cc_library(
    name = "core",
    srcs = ["core.cc"],
    hdrs = ["core.h"],
    deps = [
        "@gcc_11//:lib",  # Explicit compiler
    ],
)

# embedded/bootloader/BUILD.bazel
cc_binary(
    name = "bootloader",
    srcs = ["bootloader.c"],
    deps = [
        "@arm_none_eabi//:arm_libs",  # ARM toolchain
    ],
)

# java_apps/billing/BUILD.bazel
java_binary(
    name = "billing_service",
    srcs = glob(["src/**/*.java"]),
    deps = [
        "@maven//:org_junit_jupiter_junit_jupiter_api",  # Maven dep
        "@maven//:com_google_guava_guava",
    ],
)
```

**Advantages:**
- ✅ Clean separation by language/type
- ✅ Explicit version control per toolchain
- ✅ Easy to add new toolchain types
- ✅ Minimal MODULE.bazel (easy to read)
- ✅ Each project sees only what it needs

### Strategy 2: Platforms and Constraints

**Best for:** Projects that frequently switch between toolchains or have complex selection logic

Define constraint dimensions and let Bazel automatically select toolchains.

```starlark
# toolchains/constraints/BUILD.bazel
constraint_setting(name = "language")

constraint_value(
    name = "python",
    constraint_setting = ":language",
)

constraint_value(
    name = "cpp",
    constraint_setting = ":language",
)

constraint_value(
    name = "java",
    constraint_setting = ":language",
)

constraint_value(
    name = "embedded_c",
    constraint_setting = ":language",
)

# Define platforms for each language
platform(
    name = "python_platform",
    constraint_values = [":python"],
)

platform(
    name = "cpp_platform",
    constraint_values = [":cpp"],
)

platform(
    name = "java_platform",
    constraint_values = [":java"],
)

platform(
    name = "embedded_platform",
    constraint_values = [":embedded_c"],
)
```

```starlark
# toolchains/BUILD.bazel

# Python toolchain - only applied when language=python
toolchain(
    name = "python_toolchain",
    toolchain_type = "@rules_python//python:toolchain_type",
    toolchain = ":python_impl",
    target_compatible_with = ["//toolchains/constraints:python"],
)

# C++ toolchain - only applied when language=cpp
toolchain(
    name = "cpp_toolchain",
    toolchain_type = "@rules_cc//cc:toolchain_type",
    toolchain = ":cpp_impl",
    target_compatible_with = ["//toolchains/constraints:cpp"],
)

# Java toolchain - only applied when language=java
toolchain(
    name = "java_toolchain",
    toolchain_type = "@rules_jvm//java:toolchain_type",
    toolchain = ":java_impl",
    target_compatible_with = ["//toolchains/constraints:java"],
)

# Embedded toolchain - only applied when language=embedded_c
toolchain(
    name = "embedded_toolchain",
    toolchain_type = "@embedded_tools//toolchain:type",
    toolchain = ":embedded_impl",
    target_compatible_with = ["//toolchains/constraints:embedded_c"],
)
```

**Build with specific platform:**

```bash
# Build Python service
bazel build //python_services/auth:auth_service \
    --platforms=//toolchains:python_platform

# Build C++ library
bazel build //cpp_libraries/core:core \
    --platforms=//toolchains:cpp_platform

# Build embedded firmware
bazel build //embedded/bootloader:bootloader \
    --platforms=//toolchains:embedded_platform
```

**Advantages:**
- ✅ Automatic toolchain selection based on constraints
- ✅ Flexible: can add new constraint dimensions
- ✅ Powerful for cross-compilation scenarios
- ✅ Good for projects with multiple platform targets

### Strategy 3: Language-Specific Subdirectories

**Best for:** Smaller monorepos or when language groups are completely independent

Organize projects into language-specific directories with their own configuration.

```
my-monorepo/
├── MODULE.bazel                    # Minimal root config
├── python_ecosystem/
│   ├── MODULE.bazel               # Python-specific extensions
│   ├── services/
│   │   ├── auth/BUILD.bazel
│   │   └── billing/BUILD.bazel
│   └── libraries/
│       └── common/BUILD.bazel
├── cpp_ecosystem/
│   ├── MODULE.bazel               # C++-specific extensions
│   ├── libraries/
│   │   ├── core/BUILD.bazel
│   │   └── util/BUILD.bazel
│   └── binaries/
│       ├── app/BUILD.bazel
├── java_ecosystem/
│   ├── MODULE.bazel               # Java-specific extensions
│   ├── services/
│   │   └── billing/BUILD.bazel
│   └── libraries/
│       └── common/BUILD.bazel
└── embedded_ecosystem/
    ├── MODULE.bazel               # Embedded-specific extensions
    ├── firmware/
    │   └── bootloader/BUILD.bazel
    └── drivers/
        └── uart/BUILD.bazel
```

**Each ecosystem has its own MODULE.bazel:**

```starlark
# python_ecosystem/MODULE.bazel
module(name = "python_ecosystem", version = "1.0.0")

bazel_dep(name = "rules_python", version = "0.20.0")

python = use_extension("@rules_python//python:extensions.bzl", "python")
python.toolchain(python_version = "3.11")
use_repo(python, "python_3_11")
```

```starlark
# cpp_ecosystem/MODULE.bazel
module(name = "cpp_ecosystem", version = "1.0.0")

bazel_dep(name = "rules_cc", version = "0.1.1")

cc = use_extension("@rules_cc//cc:extensions.bzl", "cc")
cc.configure(gcc_version = "11.0")
use_repo(cc, "gcc_11")
```

**Advantages:**
- ✅ Clear separation by language
- ✅ Each ecosystem is relatively independent
- ✅ Easy to understand for newcomers
- ✅ Can optimize per-language toolchain settings

**Disadvantages:**
- ❌ Multiple MODULE.bazel files to maintain
- ❌ More complex for cross-language dependencies
- ❌ Duplicated configuration

---

## Decision Matrix

| Scenario | Strategy 1 | Strategy 2 | Strategy 3 |
|----------|-----------|-----------|-----------|
| **Many languages (5+)** | ✅ Best | ⚠️ OK | ❌ Unwieldy |
| **Frequent cross-language deps** | ✅ Best | ✅ Good | ⚠️ Complex |
| **Language groups independent** | ✅ Good | ⚠️ OK | ✅ Best |
| **Simple monorepo (2-3 languages)** | ✅ Good | ✅ Good | ✅ OK |
| **Complex platform requirements** | ✅ Good | ✅ Best | ⚠️ Difficult |
| **Maintenance overhead** | ✅ Low | ✅ Low | ⚠️ Medium |

**Recommendation:** Use **Strategy 1 (Module Extensions)** for most scenarios. It scales well, stays maintainable, and keeps the root MODULE.bazel clean.

---

## Complete Example: Mixed-Language Monorepo

Here's a production-grade setup:

```
my-monorepo/
├── MODULE.bazel
├── .bazelrc
├── python_services/
│   ├── auth/BUILD.bazel
│   └── billing/BUILD.bazel
├── cpp_libraries/
│   ├── core/BUILD.bazel
│   └── util/BUILD.bazel
├── embedded/
│   ├── bootloader/BUILD.bazel
│   └── firmware/BUILD.bazel
├── java_apps/
│   └── processor/BUILD.bazel
└── toolchains/
    ├── BUILD.bazel
    ├── python_config.bzl
    ├── cpp_config.bzl
    ├── embedded_config.bzl
    └── java_config.bzl
```

```starlark
# MODULE.bazel
module(name = "my-monorepo", version = "1.0.0")

# Dependencies
bazel_dep(name = "rules_python", version = "0.20.0")
bazel_dep(name = "rules_cc", version = "0.1.1")
bazel_dep(name = "embedded_tools", version = "1.0.0")
bazel_dep(name = "rules_jvm", version = "5.0.0")

# Load configuration from separate .bzl files
python = use_extension("//toolchains:python_config.bzl", "python_ext")
use_repo(python, "python_3_11", "python_3_10")

cc = use_extension("//toolchains:cpp_config.bzl", "cc_ext")
use_repo(cc, "gcc_11", "clang_14")

embedded = use_extension("//toolchains:embedded_config.bzl", "embedded_ext")
use_repo(embedded, "arm_none_eabi", "riscv64")

maven = use_extension("//toolchains:java_config.bzl", "maven_ext")
use_repo(maven, "maven")
```

```starlark
# toolchains/python_config.bzl
def _python_ext_impl(ctx):
    python = ctx.find_python()
    # ... setup code ...

python_ext = module_extension(
    implementation = _python_ext_impl,
)
```

```starlark
# python_services/auth/BUILD.bazel
py_binary(
    name = "auth_service",
    srcs = ["main.py"],
    deps = ["@python_3_11//:lib"],
)
```

---

## Best Practices

### ✅ Do

- Keep MODULE.bazel clean and readable (delegate config to extensions)
- Use descriptive names: `python_3_11`, `gcc_11`, not `py`, `cc`
- Document which projects use which toolchains (add comments)
- Version all external toolchains explicitly
- Use constraints when you have complex selection logic
- Test each language/toolchain in CI independently

### ❌ Don't

- Don't put all toolchain configuration inline in MODULE.bazel
- Don't force all projects to use all toolchains
- Don't mix configuration concerns (language + platform + architecture)
- Don't create a MODULE.bazel per project (breaks monorepo semantics)
- Don't assume all developers have all toolchains installed locally

---

## Transitioning to This Pattern

If you're starting with a flat monorepo:

1. **Audit current setup** — List all toolchains currently in use
2. **Choose strategy** — Use decision matrix above
3. **Implement gradually** — Migrate projects one language at a time
4. **Update CI configuration** — Ensure each CI job uses correct toolchain
5. **Document in .bazelrc** — Help developers know which platform to build with

Example `.bazelrc`:

```
# Python projects
build:python --platforms=//toolchains:python_platform
test:python --platforms=//toolchains:python_platform

# C++ projects
build:cpp --platforms=//toolchains:cpp_platform
test:cpp --platforms=//toolchains:cpp_platform

# Embedded projects
build:embedded --platforms=//toolchains:embedded_platform
test:embedded --platforms=//toolchains:embedded_platform
```

Then developers can run:
```bash
bazel build //python_services/... --config=python
bazel build //cpp_libraries/... --config=cpp
bazel build //embedded/... --config=embedded
```

---

## See Also

- [[concepts/advanced/module-extensions]] — How module extensions work
- [[reference/platforms-toolchains-rules.md]] — Platform and toolchain rules
- [[patterns/platform-configuration]] — Platform setup in detail
- [[concepts/fundamentals/module.bazel]] — MODULE.bazel format
