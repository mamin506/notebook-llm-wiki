---
title: "Migrating from CMake to Bazel"
category: "patterns"
level: "intermediate"
status: "seedling"
sources: []
tags: ["migration", "cmake", "refactoring", "tools"]
related: ["[[reference/bazel-vs-cmake]]", "[[concepts/advanced/build-systems-landscape]]", "[[concepts/fundamentals/packages]]", "[[concepts/fundamentals/module.bazel]]"]
last_updated: "2026-07-20"
---

# Migrating from CMake to Bazel

A practical guide for migrating existing CMake projects to Bazel.

## Why Migrate?

**Reasons to migrate to Bazel:**
- Monorepo with many projects needing unified builds
- Multi-language project (Python, C++, Java, etc.)
- Team grows and CI/CD time becomes critical
- Need reproducible, distributable builds
- Remote caching would save significant build time

**Reasons to stay with CMake:**
- Small to medium project with simple dependencies
- Team is highly productive with CMake
- Existing ecosystem (many find_package() solutions)
- Project is stable, rarely changes

---

## Pre-Migration Assessment

### Complexity Audit

Before starting, assess your CMake project:

```
Low Complexity (< 20 targets):
- Direct migration usually 1-2 days
- Manual translation of each CMakeLists.txt

Medium Complexity (20-100 targets):
- Structured migration 1-2 weeks
- Use Gazelle (auto-generate BUILD files where possible)
- Gradual rollout (convert high-level targets first)

High Complexity (100+ targets):
- Major effort 1-3 months
- Consider hybrid approach (CMake + Bazel coexistence)
- Prioritize high-value targets first
```

### Dependency Graph Analysis

**Questions to answer:**
1. How many targets (libraries, binaries, tests)?
2. How many external dependencies (find_package calls)?
3. How many language types (C++, Python, Java, etc.)?
4. Are there circular dependencies?
5. How are versions currently managed?

```bash
# Estimate CMake complexity
find . -name "CMakeLists.txt" | wc -l
grep -r "add_library\|add_executable" . | wc -l
grep -r "find_package" . | wc -l
```

### Dependency Mapping

Create a mapping of CMake find_package() calls to Bazel modules:

```
CMake find_package()          Bazel Module
─────────────────────────────────────────────
find_package(OpenSSL)    -->  @openssl (BCR)
find_package(Boost)      -->  @boost (BCR)
find_package(Python3)    -->  @rules_python
find_package(gtest)      -->  @com_google_googletest (BCR)
find_package(fmt)        -->  @fmt (BCR)

Internal libraries:
add_library(mylib)       -->  //lib:mylib (BUILD.bazel)
```

---

## Phase 1: Infrastructure Setup (Days 1-3)

### Step 1: Create Module and Root BUILD

**MODULE.bazel**
```starlark
module(
    name = "my-project",
    version = "1.0.0",
)

# List all dependencies (replace with your actual deps)
bazel_dep(name = "rules_cc", version = "0.1.1")
bazel_dep(name = "openssl", version = "3.1.0")
bazel_dep(name = "boost", version = "1.82.0")
bazel_dep(name = "com_google_googletest", version = "1.14.0")

# Language-specific rules if needed
bazel_dep(name = "rules_python", version = "0.20.0")

python = use_extension("@rules_python//python:extensions.bzl", "python")
python.toolchain(python_version = "3.11")
use_repo(python, "python_3_11")
```

**BUILD.bazel** (root)
```starlark
# Usually empty or contains top-level aliases
```

### Step 2: Create .bazelrc

**.bazelrc**
```
# Build configuration
build --platforms=@platforms//os:linux
build --platforms=@platforms//cpu:x86_64

# Test configuration
test --test_output=short
test --test_verbose_timeout_warnings

# Development convenience
build --workspace_status_command="echo dev"

# Optimization for CI/CD
build:ci --jobs=auto
build:ci --remote_cache=grpcs://cache.company.com
```

### Step 3: Create Directory Structure

Organize your project for Bazel:

```
# CMake structure
src/
├── CMakeLists.txt
├── lib/
│   ├── CMakeLists.txt
│   └── mylib.cc
└── bin/
    ├── CMakeLists.txt
    └── main.cc

# Refactor to Bazel structure
src/
├── lib/
│   ├── BUILD.bazel       ← Add this
│   └── mylib.cc
└── bin/
    ├── BUILD.bazel       ← Add this
    └── main.cc
```

**Do NOT delete CMakeLists.txt yet** — you'll need it as reference while building BUILD.bazel files.

---

## Phase 2: Gradual Migration (Weeks 1-4)

### Strategy: Bottom-Up Translation

**Translate highest-level targets first, work downward.**

```
CMakeLists.txt (top-level)          BUILD.bazel
────────────────────────────────────────────────
add_executable(myapp main.cc)   -->  cc_binary(
                                       name = "myapp",
                                       srcs = ["main.cc"],
                                       deps = [...],
                                     )

add_library(mylib mylib.cc)     -->  cc_library(
                                       name = "mylib",
                                       srcs = ["mylib.cc"],
                                       hdrs = ["mylib.h"],
                                     )

target_link_libraries(myapp ...) -->  deps = [...]

target_include_directories(...)  -->  includes = [...],
                                       local_includes = [...]
```

### Step 1: Translate Libraries

**CMakeLists.txt**
```cmake
add_library(mylib
    src/mylib.cc
    src/helper.cc
)

target_include_directories(mylib PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)

target_link_libraries(mylib PUBLIC
    OpenSSL::Crypto
    Boost::system
)

target_compile_options(mylib PRIVATE
    -Wall -Wextra -O2
)
```

**BUILD.bazel** (lib/)
```starlark
load("@rules_cc//cc:defs.bzl", "cc_library")

cc_library(
    name = "mylib",
    srcs = [
        "src/mylib.cc",
        "src/helper.cc",
    ],
    hdrs = glob(["include/**/*.h"]),
    includes = ["include"],
    deps = [
        "@openssl//:crypto",
        "@boost//:system",
    ],
    copts = ["-Wall", "-Wextra", "-O2"],
)
```

### Step 2: Translate Executables

**CMakeLists.txt**
```cmake
add_executable(myapp src/main.cc)

target_link_libraries(myapp PRIVATE
    mylib
    fmt::fmt
)
```

**BUILD.bazel** (bin/)
```starlark
load("@rules_cc//cc:defs.bzl", "cc_binary")

cc_binary(
    name = "myapp",
    srcs = ["src/main.cc"],
    deps = [
        "//lib:mylib",
        "@fmt//:fmt",
    ],
)
```

### Step 3: Translate Tests

**CMakeLists.txt**
```cmake
add_executable(mylib_test test/mylib_test.cc)

target_link_libraries(mylib_test PRIVATE
    mylib
    GTest::gtest_main
)

add_test(NAME mylib_test COMMAND mylib_test)
```

**BUILD.bazel** (lib/)
```starlark
load("@rules_cc//cc:defs.bzl", "cc_test")

cc_test(
    name = "mylib_test",
    srcs = ["test/mylib_test.cc"],
    deps = [
        ":mylib",
        "@com_google_googletest//:gtest_main",
    ],
)
```

### Step 4: Handle Custom Build Rules

**CMake custom rules (genrule equivalent):**

**CMakeLists.txt**
```cmake
add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/generated.cc
    COMMAND ${CMAKE_COMMAND} -E echo "// Generated" > generated.cc
    MAIN_DEPENDENCY proto.proto
)

add_library(proto_generated ${CMAKE_CURRENT_BINARY_DIR}/generated.cc)
```

**BUILD.bazel** (using genrule as fallback)
```starlark
load("@rules_cc//cc:defs.bzl", "cc_library", "genrule")

genrule(
    name = "generate_code",
    srcs = ["proto.proto"],
    outs = ["generated.cc"],
    cmd = "echo '// Generated' > $@",
)

cc_library(
    name = "proto_generated",
    srcs = [":generate_code"],
)
```

**Better: Use language-specific rules if available**
```starlark
# For Protocol Buffers
proto_library(name = "proto", srcs = ["proto.proto"])
cc_proto_library(name = "proto_cc", deps = [":proto"])

# For data files
genrule(
    name = "embed_config",
    srcs = ["config.txt"],
    outs = ["config.h"],
    cmd = "xxd -i $< $@",
)
```

---

## Phase 3: Dependency Management (Week 2-3)

### Convert find_package() to bazel_dep()

**Identify all external dependencies:**

```bash
grep -r "find_package" . | sort | uniq
```

**For each dependency, find Bazel module:**

```
1. Check Bazel Central Registry (BCR): https://registry.bazel.build
2. Search GitHub for bazel rules (rules_*) for that library
3. As last resort, use http_archive or git_repository
```

**Common mapping (BCR available):**
```starlark
# Old CMake approach
find_package(OpenSSL REQUIRED)      # Version undefined
find_package(fmt REQUIRED)          # Version undefined
find_package(nlohmann_json)         # May be installed or not

# Bazel approach
bazel_dep(name = "openssl", version = "3.1.0")
bazel_dep(name = "fmt", version = "10.1.1")
bazel_dep(name = "nlohmann_json", version = "3.11.2")
```

### Handle Version Locks

**CMake:** Versions are implicit and environment-dependent.

**Bazel:** Versions are explicit and locked:

```bash
# Generate MODULE.bazel.lock
bazel mod graph

# This creates MODULE.bazel.lock with all resolved versions
# Commit this to version control:
git add MODULE.bazel.lock
git commit -m "Lock Bazel module versions"
```

### Handle Private/Internal Dependencies

For dependencies not in BCR, use override:

```starlark
# MODULE.bazel
bazel_dep(name = "my_internal_lib", version = "1.0.0")

# Add override to use local version
local_path_override(
    module_name = "my_internal_lib",
    path = "../my_internal_lib",
)

# Or git repository:
git_override(
    module_name = "my_internal_lib",
    remote = "https://github.com/mycompany/my_internal_lib.git",
    commit = "abc123def456...",
)
```

---

## Phase 4: Incremental Testing (Week 3-4)

### Build Incrementally

**Don't rewrite everything at once.** Test incrementally:

```bash
# Test one target
bazel build //lib:mylib
bazel test //lib:mylib_test

# Once passing, move to next target
bazel build //bin:myapp
bazel test //bin:myapp_test

# Build entire subtree
bazel build //...
```

### Run Old and New Build Systems in Parallel

**During migration:**
```bash
# Old CMake build still works
cmake ..
make

# New Bazel build alongside
bazel build //...

# Compare binaries to verify correctness
diff cmake-build-debug/myapp bazel-bin/bin/myapp
```

### Cross-Check Results

```bash
# CMake build
cmake -DCMAKE_BUILD_TYPE=Release ..
make
./bin/myapp > cmake_output.txt

# Bazel build
bazel build -c opt //bin:myapp
bazel run //bin:myapp > bazel_output.txt

# Compare
diff cmake_output.txt bazel_output.txt
```

---

## Phase 5: Cleanup and Optimization (Week 4-5)

### Remove CMakeLists.txt Files

Once Bazel build fully works:

```bash
# Back up CMakeLists.txt (just in case)
find . -name "CMakeLists.txt" -exec mv {} {}.backup \;

# Remove cmake build artifacts
rm -rf cmake-build-debug cmake-build-release build/

# Verify Bazel still builds
bazel clean --expunge
bazel build //...
```

### Optimize Bazel Configuration

```starlark
# Add --config flags for common scenarios
# .bazelrc
build:dev -c fastbuild
build:release -c opt
build:asan --copt=-fsanitize=address --linkopt=-fsanitize=address

# CI/CD configuration
build:ci --jobs=auto
build:ci --remote_cache=grpcs://cache.company.com
```

### Add Bazel to CI/CD

```yaml
# .github/workflows/build.yml
- name: Build with Bazel
  run: bazel build //... --config=ci
  
- name: Test with Bazel
  run: bazel test //... --config=ci
```

---

## Common Migration Challenges

### Challenge 1: Complex Dependency Versions

**Problem:** CMake find_package() accepts any installed version. Bazel requires exact version.

**Solution:**
```starlark
# Use version constraints
bazel_dep(name = "boost", version = ">=1.82.0, <2.0.0")

# Or use Bazel overrides to test version changes
override(module_name = "boost", version = "1.83.0")
```

### Challenge 2: Conditional Compilation

**CMake:**
```cmake
if(WIN32)
    add_library(platform_win32.cc)
else()
    add_library(platform_unix.cc)
endif()
```

**Bazel:**
```starlark
# Use select() with constraints
cc_library(
    name = "platform",
    srcs = select({
        "@platforms//os:windows": ["platform_win32.cc"],
        "@platforms//os:linux": ["platform_unix.cc"],
        "//conditions:default": ["platform_unix.cc"],
    }),
)
```

### Challenge 3: Source Generation

**CMake:**
```cmake
add_custom_command(OUTPUT generated.cc ...)
```

**Bazel:**
```starlark
# Use genrule or language-specific rules
genrule(
    name = "codegen",
    srcs = ["schema.proto"],
    outs = ["generated.cc"],
    cmd = "protoc $< --cc_out=$(@D)",
)
```

### Challenge 4: Custom Toolchains

**CMake:**
```cmake
set(CMAKE_CXX_COMPILER /path/to/compiler)
```

**Bazel:**
```starlark
# MODULE.bazel
cc = use_extension("@rules_cc//cc:extensions.bzl", "cc")
cc.configure(compiler = "/path/to/compiler")
```

### Challenge 5: Multi-Language Projects

**CMake:** Complex (each language needs separate find/add setup)

**Bazel:** Natural (all languages coexist)
```starlark
# Single BUILD.bazel handles Python, C++, Java
py_binary(name = "app", ...)
cc_library(name = "ext", ...)
java_library(name = "lib", ...)
```

---

## Hybrid Approach: Coexistence

If full migration is too risky, use hybrid approach:

```bash
# Keep CMake for legacy code
cmake ..
make

# Use Bazel for new code
bazel build //new_subsystem/...

# Gradually move code from CMake to Bazel
# Over time, CMake becomes smaller and eventually removed
```

**Advantages:**
- ✅ Low risk (can rollback to CMake anytime)
- ✅ Allows incremental migration
- ✅ Team can learn Bazel gradually

**Disadvantages:**
- ❌ Maintain two build systems temporarily
- ❌ More complex CI/CD

---

## Post-Migration Checklist

- [ ] All targets build successfully with `bazel build //...`
- [ ] All tests pass with `bazel test //...`
- [ ] MODULE.bazel.lock committed to version control
- [ ] CI/CD pipeline uses `bazel build/test` commands
- [ ] .bazelrc configured for team's needs
- [ ] Remote cache configured (optional but recommended)
- [ ] CMakeLists.txt files removed or archived
- [ ] Documentation updated (CONTRIBUTING.md, etc.)
- [ ] Team trained on Bazel (see [[reference/cli-reference]])
- [ ] Performance benchmarked (compare build times)

---

## Key Takeaways

1. **Start with infrastructure** — Set up MODULE.bazel, .bazelrc, root BUILD.bazel
2. **Migrate bottom-up** — Libraries first, then executables, then tests
3. **Keep old build temporarily** — Compare outputs for correctness
4. **Test incrementally** — Don't rewrite everything at once
5. **Lock versions explicitly** — Commit MODULE.bazel.lock
6. **Consider hybrid approach** — If full migration too risky

---

## See Also

- [[reference/bazel-vs-cmake]] — Detailed comparison
- [[concepts/fundamentals/packages]] — Bazel package structure
- [[concepts/fundamentals/module.bazel]] — MODULE.bazel format
- [[reference/cli-reference]] — Bazel commands
