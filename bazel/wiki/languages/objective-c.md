---
title: "Objective-C in Bazel"
category: "languages"
level: "intermediate"
status: "seedling"
sources: ["Objective-C Rules.md"]
tags: ["#objective-c", "#ios", "#macos", "#apple", "#language-specific"]
related: ["[[languages/cpp]]", "[[reference/general-rules]]", "[[concepts/advanced/platforms]]"]
last_updated: "2026-07-19"
graph-group: "languages"
---

# Objective-C in Bazel

Bazel provides rules for building Objective-C and Objective-C++ code targeting iOS, macOS, and other Apple platforms.

## Core Objective-C Rules

### objc_library

A reusable Objective-C library.

**Key Attributes:**
- `srcs`: Objective-C source files (`.m`, `.mm` for Objective-C++, `.c`, `.cc`)
- `hdrs`: Public header files (`.h`)
- `non_arc_srcs`: Source files that do NOT use Automatic Reference Counting (ARC)
- `deps`: Other `objc_library` targets
- `data`: Runtime data files
- `includes`: Include directories (passed to dependents)
- `sdk_frameworks`: Apple frameworks to link (e.g., "UIKit", "QuartzCore")
- `sdk_dylibs`: System libraries to link (e.g., "libz", "libarchive")
- `weak_sdk_frameworks`: Frameworks to link weakly (symbols may not be present at runtime)
- `defines`: Preprocessor defines (passed to this library and dependents)
- `local_defines`: Preprocessor defines (this library only)
- `copts`: Compiler options
- `enable_modules`: Enable Clang module support (`@import` statements)
- `module_name`: Custom module name (default: target path with underscores)

**Example:**
```python
objc_library(
    name = "ui_lib",
    srcs = ["UIView+Extensions.m"],
    hdrs = ["UIView+Extensions.h"],
    deps = [":base_lib"],
    sdk_frameworks = ["UIKit"],
)
```

### objc_import

Imports pre-compiled Objective-C libraries (`.a` files).

**Key Attributes:**
- `archives`: Pre-compiled static library files (`.a`)
- `hdrs`: Header files for the library
- `sdk_frameworks`: Apple frameworks to link
- `sdk_dylibs`: System libraries to link
- `includes`: Include directories
- `alwayslink`: Force all object files to be linked

**Example:**
```python
objc_import(
    name = "third_party_lib",
    hdrs = glob(["include/**/*.h"]),
    archives = ["lib/libthirdparty.a"],
    includes = ["include"],
    sdk_frameworks = ["CoreFoundation"],
)
```

## Key Concepts

### Automatic Reference Counting (ARC)

By default, all Objective-C source files in `srcs` are compiled with ARC enabled. To compile specific files without ARC:

```python
objc_library(
    name = "mixed_arc_lib",
    srcs = ["modern_code.m"],  # With ARC
    non_arc_srcs = ["legacy_code.m"],  # Without ARC
    hdrs = ["mixed_arc_lib.h"],
)
```

### Apple Frameworks

Link against Apple system frameworks using `sdk_frameworks`:

```python
objc_library(
    name = "camera_lib",
    srcs = ["Camera.m"],
    hdrs = ["Camera.h"],
    sdk_frameworks = [
        "AVFoundation",  # Camera framework
        "CoreGraphics",  # Graphics
        "UIKit",         # UIView, etc.
    ],
)
```

### Clang Modules

Enable C++20-style module support:

```python
objc_library(
    name = "modular_lib",
    srcs = ["module_code.m"],
    hdrs = ["ModuleHeader.h"],
    enable_modules = True,
)
```

With modules enabled, you can use:
```objectivec
@import UIKit;
@import path_to_package_target;
```

### Header Search Paths

Control how headers are found during compilation:

```python
objc_library(
    name = "lib",
    srcs = ["lib.m"],
    hdrs = ["lib.h"],
    includes = ["include"],  # Add include/ to search path for this lib and dependents
)
```

## Common Patterns

### Multi-File Framework

```
foundation/
├── BUILD
├── public/
│   ├── Foundation.h
│   └── Foundation+Private.h
├── internal/
│   ├── impl.m
│   └── impl_private.h
└── utils.m

# BUILD:
objc_library(
    name = "foundation",
    srcs = ["utils.m", "internal/impl.m"],
    hdrs = [
        "public/Foundation.h",
        "public/Foundation+Private.h",
        "internal/impl_private.h",  # Private to this library
    ],
)
```

### Mixed ARC and Non-ARC Code

```python
objc_library(
    name = "legacy_wrapper",
    srcs = ["modern_wrapper.m"],  # Modern, uses ARC
    non_arc_srcs = ["legacy_code.m"],  # Old code, manual reference counting
    hdrs = ["legacy_wrapper.h"],
    deps = [":base_lib"],
)
```

### Third-Party Pre-Compiled Library

```python
objc_import(
    name = "analytics_sdk",
    archives = ["vendor/analytics/libAnalytics.a"],
    hdrs = glob(["vendor/analytics/include/**/*.h"]),
    includes = ["vendor/analytics/include"],
    sdk_frameworks = ["CoreTelephony"],
    alwayslink = True,  # Force linking even if symbols seem unused
)

objc_library(
    name = "app",
    srcs = ["app.m"],
    hdrs = ["app.h"],
    deps = [":analytics_sdk"],
)
```

## Compilation Options

### Compiler Flags

```python
objc_library(
    name = "strict_lib",
    srcs = ["lib.m"],
    hdrs = ["lib.h"],
    copts = [
        "-Wall",
        "-Werror",
        "-fno-objc-arc",  # Disable ARC for specific target
    ],
)
```

### Preprocessor Defines

```python
objc_library(
    name = "config_lib",
    srcs = ["lib.m"],
    defines = ["VERSION=1.0", "DEBUG"],  # Affects this and dependents
    local_defines = ["INTERNAL_FLAG"],  # Affects only this library
)
```

### SDK Includes

For headers in the SDK, use `sdk_includes`:

```python
objc_library(
    name = "sdk_wrapper",
    srcs = ["wrapper.m"],
    sdk_includes = ["System/Library/Frameworks/CoreFoundation.framework/Headers"],
)
```

## Testing Objective-C Code

Create tests using `objc_library` with test-specific configurations:

```python
objc_library(
    name = "lib",
    srcs = ["lib.m"],
    hdrs = ["lib.h"],
)

objc_library(
    name = "lib_test",
    srcs = ["lib_test.m"],
    deps = [":lib"],
    testonly = True,  # Mark as test-only
)
```

For XCTest integration, link against the XCTest framework:

```python
objc_library(
    name = "unit_test",
    srcs = ["unit_test.m"],
    deps = [":lib"],
    sdk_frameworks = ["XCTest"],
    testonly = True,
)
```

## Common Issues

**Issue: "undefined reference to symbol"**
- Add missing framework to `sdk_frameworks` or `sdk_dylibs`
- Ensure dependency is in `deps`
- Check if library should use `alwayslink=True`

**Issue: "include file not found"**
- Verify include path in `includes` or `sdk_includes`
- Check that header file is in `hdrs` (public) or `srcs` (private)
- Ensure dependent library is listed in `deps`

**Issue: "ARC forbidden" error**
- Use `non_arc_srcs` for files that shouldn't use ARC
- Or add `-fno-objc-arc` to `copts` if needed globally

**Issue: "framework not found"**
- Use correct framework name in `sdk_frameworks` (e.g., "UIKit", not "uikit")
- Ensure target platform supports the framework (iOS vs macOS differences)
- Check that platform constraints are set correctly

## Platform-Specific Considerations

Objective-C is primarily used on Apple platforms (iOS, macOS, tvOS, watchOS). When using Bazel with Objective-C, ensure your platform constraints and toolchain are configured for the target platform.

See [[concepts/advanced/platforms]] for platform configuration and [[reference/build-options]] for build flags specific to Apple targets.

## See Also

- [[languages/cpp]] — C++ in Bazel (similar rules and concepts)
- [[concepts/advanced/platforms]] — Platform and toolchain configuration
- [[reference/build-options]] — Build options and flags
- [[patterns/dependency-management]] — Dependency patterns
