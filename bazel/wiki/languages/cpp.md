---
title: "C++ in Bazel"
category: "languages"
level: "fundamentals"
status: "growing"
sources: ["C  C++ Rules.md"]
tags: ["#cpp", "#c++", "#rules", "#language-specific"]
related: ["[[concepts/fundamentals/targets]]", "[[reference/general-rules]]", "[[patterns/dependency-management]]"]
last_updated: "2026-07-19"
graph-group: "languages"
---

# C++ in Bazel

C++ support in Bazel provides hermetic, reproducible compilation with powerful linking and dependency management through the `rules_cc` ruleset.

## Core C++ Rules

### cc_library

A reusable C++ library (object files, static `.a`, or shared `.so`).

**Key Attributes:**
- `srcs`: C/C++ source files (`.c`, `.cc`, `.cpp`, `.cxx`)
  - These are the files that get compiled
  - `.h` files in `srcs` are private to the library
- `hdrs`: Public header files (`.h`, `.hh`, `.hpp`)
  - These are the library's interface; consumers can `#include` them
  - Consumers can include `hdrs` but NOT `srcs` headers
- `deps`: Other `cc_library` or `cc_import` targets this library depends on
- `data`: Runtime data files
- `defines`: Preprocessor defines applied to this lib and dependents (use sparingly)
- `local_defines`: Preprocessor defines applied to this lib only
- `includes`: Include directories for this lib and dependents (via `-isystem`)
- `local_includes`: Include directories for this lib only (via `-I`)
- `copts`: Compiler options (C/C++)
- `cxxopts`: Compiler options (C++ only)
- `conlyopts`: Compiler options (C only)
- `linkopts`: Linker options (applied when linking dependents)
- `linkstatic`: `True` (default) = static linking (`.a`), `False` = dynamic (`.so`)

**Example:**
```python
cc_library(
    name = "mylib",
    srcs = ["mylib.cc"],
    hdrs = ["mylib.h"],
    deps = [":helper"],
    copts = ["-O2"],
    linkopts = ["-lpthread"],
)
```

**Header Inclusion Rules:**
- Public headers (`hdrs`) can include:
  - Other headers in `hdrs`
  - Private headers from other `cc_library` deps (via `#include`)
- Private headers (`srcs`) can include:
  - Public and private headers from this library
  - Public headers from dependencies
- Consumers of the library can include only public headers (`hdrs`)

### cc_binary

Produces a linkable C++ executable.

**Key Attributes:**
- `srcs`: C/C++ source files (includes your `main()`)
- `deps`: Libraries to link (usually `cc_library` targets)
- `data`: Files available at runtime
- `linkopts`: Linker flags
- `linkstatic`: `True` (default) = static linking, `False` = dynamic
- `stamp`: Whether to embed build metadata (commit, timestamp) in the binary

**Example:**
```python
cc_binary(
    name = "myapp",
    srcs = ["main.cc", "main.h"],
    deps = [":mylib"],
    linkstatic = True,  # Prefer static linking
)
```

### cc_test

Runs C++ unit tests.

**Key Attributes:**
- Similar to `cc_binary`
- Bazel automatically links against gtest/googletest if available
- Inherits test attributes: `size`, `timeout`, `tags`, `flaky`

**Example:**
```python
cc_test(
    name = "mylib_test",
    srcs = ["mylib_test.cc"],
    deps = [
        ":mylib",
        "@com_google_googletest//:gtest_main",
    ],
)
```

### cc_import

Imports pre-compiled C++ libraries (useful for third-party code without BUILD files).

**Key Attributes:**
- `hdrs`: Header files for the precompiled lib
- `static_library`: Path to `.a` or `.lib` file
- `shared_library`: Path to `.so`, `.dll`, or `.dylib` file
- `interface_library`: For Windows `.dll` / Unix `.ifso` interface libraries
- `system_provided`: `True` if the lib is provided by the OS/system (don't expect Bazel to make it available)

**Example:**
```python
cc_import(
    name = "curl",
    hdrs = glob(["include/curl/*.h"]),
    static_library = "lib/libcurl.a",
    includes = ["include"],
)
```

### cc_shared_library

Combines multiple `cc_library` targets into a single shared library (`.so`/`.dll`).

**Attributes:**
- `deps`: Libraries to include in the shared library
- `dynamic_deps`: Other shared libraries this one depends on (will NOT be statically linked)
- `roots`: Export lists (which symbols are visible outside the library)

**Example:**
```python
cc_shared_library(
    name = "mylib_shared",
    deps = [":mylib", ":helper"],
    dynamic_deps = [":other_shared"],
)
```

## Compilation Modes

Control how C++ is compiled:

- `fastbuild` (default): No optimization, debugging symbols included
- `dbg`: Optimization off, debugging symbols included
- `opt`: Maximum optimization, minimal debugging symbols

Use `bazel build -c opt //myapp` to build in opt mode.

## Linking Strategies

**Static Linking** (`linkstatic=True`):
- Embeds all dependent libraries into the binary
- Produces larger binaries but no runtime dependencies
- Fully reproducible and hermetic

**Dynamic Linking** (`linkstatic=False`):
- Links against shared libraries (`.so`)
- Smaller binary but requires dependencies at runtime
- Less deterministic (depends on system libraries)

**Fully Static** (feature: `fully_static_link`):
- Links everything statically, even system libraries (mostly)
- Used in release builds for portable binaries

## Advanced Features

### Header Inclusion Checking

Enable `layering_check` feature to enforce header visibility:

```python
package(features = ["layering_check"])

cc_library(
    name = "mylib",
    hdrs = ["mylib.h"],
    srcs = ["mylib_impl.h", "mylib.cc"],  # impl headers stay private
)
```

Consumers can only `#include "mylib.h"`, not `"mylib_impl.h"`.

### C++20 Modules

Use `module_interfaces` attribute to specify C++20 module interface units (`.cppm`, `.ixx`):

```python
cc_library(
    name = "mylib",
    module_interfaces = ["mylib.cppm"],
    srcs = ["mylib.cc"],
)
```

Requires `--experimental_cpp_modules` flag.

### Precompiled Dependencies

If you have a pre-built library (e.g., OpenSSL), use `cc_import`:

```python
cc_import(
    name = "openssl",
    hdrs = glob(["openssl/**/*.h"]),
    static_library = "lib/libssl.a",
    static_library = "lib/libcrypto.a",
    includes = ["include"],
)

cc_binary(
    name = "app",
    srcs = ["app.cc"],
    deps = [":openssl"],
)
```

### Stamping (Build Info)

Embed build info (commit, timestamp) in binaries:

```python
cc_binary(
    name = "myapp",
    srcs = ["main.cc"],
    stamp = 1,  # Always stamp (even without --stamp)
    deps = [
        "@bazel_tools//tools/cpp:build_info_gen",
    ],
)
```

Access via link-time constants (see Bazel build info docs).

## Common Patterns

### Multi-File Library

```
mylib/
├── BUILD
├── public/
│   └── mylib.h
├── internal/
│   └── impl.h
└── mylib.cc

# BUILD:
cc_library(
    name = "mylib",
    srcs = [
        "mylib.cc",
        "internal/impl.h",
    ],
    hdrs = ["public/mylib.h"],
    include_prefix = "mylib",  # Include as "mylib/mylib.h"
)
```

### Dependency on System Library

```python
cc_library(
    name = "pthread_wrapper",
    srcs = ["pthread_wrapper.cc"],
    hdrs = ["pthread_wrapper.h"],
    linkopts = ["-lpthread"],  # Link against system libpthread
)
```

### Testing with gtest

```python
cc_test(
    name = "mylib_test",
    srcs = ["mylib_test.cc"],
    deps = [
        ":mylib",
        "@com_google_googletest//:gtest_main",
    ],
)
```

## Build Flags

- `--copt=-O3`: Add `-O3` to all C++ compiles
- `--cxxopt=-std=c++17`: Use C++17 standard
- `-c opt`: Build in optimized mode
- `--incompatible_strict_action_env`: Sandbox compilation (recommended for hermeticity)

## Common Issues

**Issue: "undefined reference to symbol"**
- Missing dependency in `deps`
- Missing `-l` flag in `linkopts` for a system library
- Linking order problem (rearrange `deps` order)

**Issue: "multiple definitions of symbol"**
- Two libraries in the dependency graph define the same symbol
- Use `cc_shared_library` to ensure symbol is in only one place
- Or tag library with `LINKABLE_MORE_THAN_ONCE` (risky; use sparingly)

**Issue: "fatal error: unknown file type with suffix .a"**
- Missing `-lstdc++` or toolchain misconfiguration
- Check that toolchain is set up correctly for your platform

## See Also

- [[patterns/dependency-management]] — Managing C++ dependencies
- [[reference/build-options]] — Build options and flags
- [[concepts/fundamentals/targets]] — Understanding Bazel targets
