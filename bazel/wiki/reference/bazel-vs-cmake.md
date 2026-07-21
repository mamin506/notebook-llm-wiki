---
title: "Bazel vs CMake: Detailed Comparison"
category: "reference"
level: "intermediate"
status: "seedling"
sources: []
tags: ["build-systems", "comparison", "tools", "architecture"]
related: ["[[concepts/advanced/build-systems-landscape]]", "[[patterns/cmake-to-bazel-migration]]", "[[concepts/advanced/hermeticity]]"]
last_updated: "2026-07-20"
---

# Bazel vs CMake: Detailed Comparison

A comprehensive comparison of Bazel and CMake, two fundamentally different approaches to building software projects.

## Executive Summary

| Aspect | Bazel | CMake |
|--------|-------|-------|
| **Design Goal** | Reproducible, distributed, efficient builds | Portable, cross-platform configuration |
| **Build Model** | Direct graph-based execution | Configuration → generate native files |
| **Language** | Starlark (restricted Python) | CMake scripting language |
| **Dependency Isolation** | Strong (hermetic by design) | Weak (environment-dependent) |
| **Caching** | Built-in local + remote | External tools (ccache) |
| **Best Fit** | Large monorepos, multi-language | Libraries, medium projects |
| **Learning Curve** | Steep (new concepts) | Gradual (scripting-like) |

---

## Architecture: Fundamental Differences

### CMake: Configuration + Generation

```
CMakeLists.txt (declarative config)
         ↓
CMake processor (interprets, generates)
         ↓
Makefile / Ninja / Visual Studio project
         ↓
make / ninja / Visual Studio (executes)
         ↓
Binary
```

**What CMake does:**
1. Reads `CMakeLists.txt` files
2. Tests the environment (compiler, libraries, system capabilities)
3. Generates native build files (Makefile, Ninja, VS projects)
4. Exits (developer runs make/ninja/etc separately)

**Implications:**
- Generated build files are specific to one machine
- Each developer may regenerate different files if environment differs
- Build is reproducible *only if* all developers have identical environments

### Bazel: Direct Build Orchestration

```
BUILD / MODULE.bazel files (declarative rules)
         ↓
Bazel analyzer (builds dependency graph, no code generation)
         ↓
Bazel executor (runs actions in parallel)
         ↓
Binary (cached locally and remotely)
```

**What Bazel does:**
1. Reads BUILD and MODULE.bazel files
2. Constructs complete dependency graph (all languages together)
3. Directly executes build actions (compiles, links, tests)
4. Caches results (local disk, remote servers)

**Implications:**
- No machine-specific generated files
- Same build files work identically on all machines
- Caching works across team (same source = same binary)

---

## Dependency Management

### CMake: Environment-Based

```cmake
# CMakeLists.txt
find_package(OpenSSL REQUIRED)           # Looks in system paths
find_package(Boost 1.70 REQUIRED)        # Finds whatever is installed
target_link_libraries(myapp OpenSSL::Crypto Boost::system)
```

**How it works:**
- `find_package()` searches standard system locations
- Depends on correct version being installed on developer's machine
- Version conflicts can cause silent bugs (wrong version silently linked)

**Problems:**
- ❌ Different machines may have different library versions
- ❌ CI/CD must replicate exact developer environment
- ❌ Hard to test multiple versions simultaneously
- ❌ "It works on my machine" syndrome

### Bazel: Explicit, Versioned Dependencies

```starlark
# MODULE.bazel
bazel_dep(name = "openssl", version = "3.1.0")
bazel_dep(name = "boost", version = "1.82.0")

# MODULE.bazel.lock (auto-generated, checked into git)
# openssl 3.1.0
# boost 1.82.0
# ... (transitive deps also locked)

# BUILD.bazel
cc_binary(
    name = "myapp",
    deps = ["@openssl//:crypto", "@boost//:system"],
)
```

**How it works:**
- All versions explicitly declared in MODULE.bazel
- Module resolution produces MODULE.bazel.lock (deterministic)
- Lock file checked into version control
- Same lock file → same binaries on all machines

**Advantages:**
- ✅ Reproducible across all machines
- ✅ Version conflicts caught at analysis time
- ✅ Easy to test multiple versions (different targets, different versions)
- ✅ Remote cache works globally (same inputs → same outputs)

---

## Dependency Isolation (Hermeticity)

### CMake: System-Dependent

```cmake
# CMakeLists.txt
find_package(Python REQUIRED)
target_include_directories(mylib PUBLIC /usr/include)  # Hardcoded path!

# If system changes (Python upgrade, paths move) → build breaks
```

**System Environment Affects Build:**
```bash
# Developer A
$ cmake ..
Found Python 3.10 at /usr/bin/python3
$ make
# Works fine

# Developer B (upgraded Python)
$ cmake ..
Found Python 3.12 at /usr/bin/python3
$ make
# Same source, different binary! Cache doesn't help.
```

### Bazel: Isolated and Deterministic

```starlark
# MODULE.bazel
bazel_dep(name = "python", version = "3.11.0")

# BUILD.bazel
py_binary(
    name = "myapp",
    srcs = ["app.py"],
    deps = ["@python//:lib"],
)
```

**Build is Isolated:**
```bash
# Developer A
$ bazel build //myapp
# Uses Python 3.11 from Bazel cache
# → Binary X

# Developer B (any Python version on system)
$ bazel build //myapp
# Uses Python 3.11 from Bazel cache
# → Identical Binary X

# CI/CD
$ bazel build //myapp
# Uses Python 3.11 from Bazel cache
# → Identical Binary X (can reuse from remote cache!)
```

**Key Principle:** All build inputs (compilers, libraries, interpreters) are explicitly declared and versioned. Build doesn't depend on system state.

---

## Multi-Language Projects

### CMake: Per-Language Configuration

```cmake
# CMakeLists.txt
find_package(Python3 REQUIRED)
find_package(SWIG REQUIRED)

# Configure Python
pybind11_add_module(myext src/bindings.cpp)

# Configure C++
add_library(mylib src/lib.cc)

# Configure Java
find_package(Java REQUIRED)
add_jar(myjar src/MyClass.java)

# Each language has separate config pipeline
```

**Problems:**
- ❌ Each language needs separate find_package() call
- ❌ Mixed-language dependencies hard to express
- ❌ Version coordination between languages manual
- ❌ No unified build model

### Bazel: Unified Multi-Language Model

```starlark
# MODULE.bazel
bazel_dep(name = "rules_python", version = "0.20.0")
bazel_dep(name = "rules_cc", version = "0.1.1")
bazel_dep(name = "rules_jvm", version = "5.0.0")

# BUILD.bazel - all languages use same rule structure
py_binary(
    name = "app",
    srcs = ["app.py"],
    deps = [":cpp_ext", ":java_lib"],
)

cc_library(
    name = "cpp_ext",
    srcs = ["ext.cc"],
    hdrs = ["ext.h"],
)

java_library(
    name = "java_lib",
    srcs = ["Lib.java"],
)
```

**Advantages:**
- ✅ All languages coexist naturally
- ✅ Cross-language dependencies straightforward
- ✅ Unified dependency resolution across languages
- ✅ One build command works for everything

---

## Incremental Builds and Caching

### CMake: File-Time Based

```bash
# CMake + Make: uses file modification times
$ touch src/main.cc
$ make
# Recompiles main.cc because file timestamp changed
# Relinks binaries that depend on main.cc
# But: ccache (external) helps with some redundancy

# Problem: if you checkout an old branch and back, timestamps get invalidated
```

**Limitations:**
- ❌ No content-addressed caching (cache hit/miss based on file paths, not content)
- ❌ ccache is external tool (optional, requires separate setup)
- ❌ No remote cache sharing (each machine's ccache is independent)
- ❌ Caching doesn't work well across CI/CD and developer machines

### Bazel: Content-Addressed Caching

```bash
$ bazel build //app:main
# Analyzes all inputs (sources, compiler version, flags)
# Computes content hash → action cache key
# Checks local cache first, then remote cache

# Same source code on any machine:
$ bazel build //app:main
# Identical hash → finds in remote cache → reuses binary!

# Checkout old branch:
$ git checkout v1.0.0
$ bazel build //app:main
# Same source as v1.0.0 → same hash → finds cached binary

# Switch back to main:
$ git checkout main
$ bazel build //app:main
# Cache hit for main's binaries (never recompiled)
```

**Built-in Caching:**
```starlark
# No configuration needed! Just works.

# Remote cache (team shares builds)
bazel build //... --remote_cache=grpcs://cache.company.com
```

**Advantages:**
- ✅ Content-addressed (same inputs → reuse forever)
- ✅ Local cache built-in
- ✅ Remote cache for team (CI/CD results automatically cached)
- ✅ Survives branch switching, rebasing, etc.

---

## Distributed and Remote Execution

### CMake: Not Supported Natively

```cmake
# No way to distribute build to remote machines
# If you want distributed builds:
# 1. Use distcc or similar (complex setup)
# 2. Or use different build system (icecc, BuildBarn)
# 3. Manual orchestration
```

### Bazel: Native RBE Support

```starlark
# BUILD.bazel - no changes needed!
cc_binary(name = "app", srcs = [...], deps = [...])

# Configure remote execution
# bazel build //app:main \
#   --remote_executor=grpcs://build-farm.company.com
```

**Execution Model:**
- ✅ Actions automatically sent to remote workers
- ✅ Local cache + remote cache both consulted
- ✅ Scales to 100s of parallel workers
- ✅ Same binaries whether built locally or remotely

---

## Reproducibility

### CMake: Difficult

```cmake
# Multiple sources of non-determinism:
# 1. System time (if cmake uses timestamps)
# 2. Environment variables
# 3. Compiler version (if find_package finds different version)
# 4. Linker flags (if system-dependent)

# Hard to verify builds are reproducible
```

**Challenge:** Two builds from same source may produce different binaries.

### Bazel: Built-in

```starlark
# Bazel enforces reproducibility by design:
# 1. All inputs explicitly declared
# 2. Timestamps stripped from binaries
# 3. Linker flags deterministic
# 4. Action graph generated identically every time

# Verify reproducibility:
$ bazel clean
$ bazel build //app:main
$ md5sum bazel-bin/app/main
# 1a2b3c4d5e6f...

$ bazel clean
$ bazel build //app:main
$ md5sum bazel-bin/app/main
# 1a2b3c4d5e6f... (same!)
```

**Reproducibility Guarantees:**
- ✅ Same source always produces same binary
- ✅ Different machines produce same binary
- ✅ Different times produce same binary
- ✅ Verifiable and testable

---

## Learning Curve

### CMake: Familiar but Complex

**Pros:**
- ✅ Looks like a scripting language
- ✅ Easy to understand first example
- ✅ Straightforward for simple projects

**Cons:**
- ❌ Growing complexity as project grows
- ❌ Many subtle behaviors (variable scoping, list handling)
- ❌ Hard to predict behavior with complex conditions

### Bazel: Steep But Pays Off

**Pros:**
- ✅ Starlark is a restricted, simpler language
- ✅ Rules hide complexity (you don't think about compiler invocation)
- ✅ More predictable behavior

**Cons:**
- ❌ New concepts: packages, targets, labels, rules
- ❌ First project takes longer to set up
- ❌ Error messages can be cryptic
- ❌ Fewer online tutorials than CMake

**Cost-Benefit:**
- Small project (< 5 targets): CMake faster to set up
- Medium project (10-100 targets): Break-even point
- Large project (100+ targets): Bazel pays dividends

---

## Cross-Platform Support

### CMake: Native Support

```cmake
# One CMakeLists.txt generates:
cmake -G "Unix Makefiles" ..    # Linux/macOS
cmake -G "Visual Studio 16" ..  # Windows
cmake -G "Xcode" ..             # macOS Xcode
```

**Mature platform support:**
- ✅ Windows, Linux, macOS (native)
- ✅ Various build tools (Make, Ninja, Visual Studio, Xcode)

### Bazel: Unified Configuration

```starlark
# One BUILD.bazel file works on all platforms
cc_binary(
    name = "app",
    srcs = ["main.cc"],
    deps = [...],
)

# No platform-specific configuration needed
# (constraints used when platform-specific behavior is needed)
```

**Platform support:**
- ✅ Windows, Linux, macOS (native)
- ✅ iOS, Android (via rules)
- ✅ Hermetic cross-compilation (easy to target different architectures)

---

## Real-World Example: Python + C++ Project

### CMake Version

**Directory structure:**
```
my-project/
├── CMakeLists.txt
├── src/
│   ├── CMakeLists.txt
│   ├── helper.cc
│   └── helper.h
└── python/
    ├── app.py
    └── setup.py
```

**CMakeLists.txt:**
```cmake
cmake_minimum_required(VERSION 3.20)
project(my-project)

# Find and configure Python
find_package(Python3 REQUIRED COMPONENTS Interpreter Development)
find_package(pybind11 REQUIRED)

# Add C++ library
add_library(helper src/helper.cc)

# Add Python extension
pybind11_add_module(helper_ext src/bindings.cpp)
target_link_libraries(helper_ext PRIVATE helper)

# Install Python module
install(FILES python/app.py DESTINATION bin)
```

**Problems:**
- ❌ Must have correct Python version installed
- ❌ Must have pybind11 installed
- ❌ Build may fail on another machine with different environment
- ❌ No deterministic version selection

### Bazel Version

**Directory structure:**
```
my-project/
├── MODULE.bazel
├── BUILD.bazel
├── cpp/
│   ├── BUILD.bazel
│   ├── helper.cc
│   └── helper.h
└── python/
    ├── BUILD.bazel
    └── app.py
```

**MODULE.bazel:**
```starlark
module(name = "my-project", version = "1.0.0")

bazel_dep(name = "rules_python", version = "0.20.0")
bazel_dep(name = "rules_cc", version = "0.1.1")
bazel_dep(name = "pybind11_bazel", version = "2.11.1.bzlmod")

python = use_extension("@rules_python//python:extensions.bzl", "python")
python.toolchain(python_version = "3.11")
use_repo(python, "python_3_11")
```

**cpp/BUILD.bazel:**
```starlark
load("@rules_cc//cc:defs.bzl", "cc_library")
load("@pybind11_bazel//:build_defs.bzl", "pybind11_extension")

cc_library(
    name = "helper",
    srcs = ["helper.cc"],
    hdrs = ["helper.h"],
)

pybind11_extension(
    name = "helper_ext",
    srcs = ["bindings.cpp"],
    deps = [":helper"],
)
```

**python/BUILD.bazel:**
```starlark
load("@rules_python//python:defs.bzl", "py_binary")

py_binary(
    name = "app",
    srcs = ["app.py"],
    data = ["//cpp:helper_ext"],
)
```

**Advantages:**
- ✅ All versions locked in MODULE.bazel.lock
- ✅ Works identically on all machines (no environment dependency)
- ✅ CI/CD produces same binaries every time
- ✅ Remote cache shares binaries across team

---

## Decision Matrix

### Use CMake If:

| Scenario | Reason |
|----------|--------|
| Small to medium C/C++ library | Fast to set up, mature ecosystem |
| Simple dependencies | Find_package() sufficient |
| Cross-platform GUI app | Native Visual Studio/Xcode support |
| Existing CMake ecosystem | Reuse existing infrastructure |

### Use Bazel If:

| Scenario | Reason |
|----------|--------|
| Monorepo with 100+ projects | Unified build, efficient caching |
| Multi-language (Python + C++) | Seamless integration |
| Large team sharing builds | Remote cache saves huge time |
| Strict reproducibility needed | Hermetic by design |
| Distributed CI/CD | RBE native support |
| Frequent branch switching | Content-based cache survives rebasing |

---

## Migration Path

See [[patterns/cmake-to-bazel-migration]] for step-by-step guidance on migrating CMake projects to Bazel.

---

## See Also

- [[concepts/advanced/build-systems-landscape]] — Overview of other build systems
- [[concepts/advanced/hermeticity]] — Why isolation matters
- [[patterns/cmake-to-bazel-migration]] — How to migrate from CMake
