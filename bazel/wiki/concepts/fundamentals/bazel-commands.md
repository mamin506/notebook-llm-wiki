---
title: "Bazel Commands: A Beginner's Guide"
category: "concepts"
level: "fundamentals"
status: "seedling"
sources: ["CLI Reference (synthesized from query skill)"]
tags: ["commands", "cli", "getting-started", "workflow"]
related: ["[[reference/cli-reference]]", "[[reference/build-options]]", "[[concepts/fundamentals/targets]]", "[[reference/calling-bazel-from-scripts]]"]
last_updated: "2026-07-20"
---

# Bazel Commands: A Beginner's Guide

This guide introduces Bazel's main commands and when to use them. For complete reference, see [[reference/cli-reference]].

---

## The Five Essential Commands

If you only learn five Bazel commands, these are them:

| Command | What It Does | When to Use |
|---------|--------------|-------------|
| **`bazel build`** | Compile targets | You want compiled artifacts |
| **`bazel test`** | Run tests | You want to verify code correctness |
| **`bazel run`** | Build and execute | You want to run a program immediately |
| **`bazel query`** | Explore the build graph | You want to understand dependencies |
| **`bazel clean`** | Remove build artifacts | You want to reset/free disk space |

---

## Command Categories

### 🏗️ Building & Running

These commands execute actual build actions.

#### `bazel build`

**Compiles targets without running them.**

```bash
# Build a single target
bazel build //app:main
# Produces: bazel-bin/app/main (executable)

# Build all targets in a directory
bazel build //app/...
# Builds every target under //app/

# Build with optimization
bazel build -c opt //app:main
# Compile with optimizations for release
```

**Use when:** You want compiled binaries/libraries but don't need to run them.

**Common options:**
- `-c opt` — Optimize for speed (release builds)
- `-c dbg` — Keep debug symbols (debugging)
- `-j 4` — Use 4 parallel jobs
- `-k` — Keep going even if some targets fail

---

#### `bazel test`

**Builds and runs tests.**

```bash
# Run all tests in a directory
bazel test //app/...
# Builds and executes all test targets

# Run specific test
bazel test //app:my_test

# Run with custom filter
bazel test --test_filter="TestClass#method" //app

# Run each test 3 times
bazel test --runs_per_test=3 //app:tests
```

**Use when:** You want to verify that code works correctly.

**Common options:**
- `--test_filter=pattern` — Only run tests matching pattern
- `--test_size_filters=small,medium` — Only run small/medium tests
- `--cache_test_results=no` — Don't cache; re-run every time
- `--test_output=all` — Show full output (default: summary only)

---

#### `bazel run` ⭐

**Builds and immediately executes a single target.**

```bash
# Build and run
bazel run //app:main
# Much faster than: bazel build //app:main && ./bazel-bin/app/main

# Run with arguments (note the --)
bazel run //app:main -- --verbose --config=prod
# Arguments after -- are passed to the program

# Run a script with file arguments
bazel run //tools:formatter -- src/main.cc
# Automatically handles runfiles and dependencies
```

**Use when:** You want to build and execute something immediately.

**Why prefer `bazel run` over manual execution:**
- ✅ Automatically handles `runfiles` (data dependencies)
- ✅ Works with remote builds (no manual path lookup)
- ✅ Rebuilds only if inputs changed
- ✅ Cleaner than copying binaries

**Common scenarios:**
```bash
# Run your app in development
bazel run //src:app

# Run a test manually (not through bazel test)
bazel run //src:my_test

# Run a code formatter
bazel run //tools:clang_format -- -i src/**/*.cc

# Run a code generator
bazel run //gen:proto_gen -- --output=gen/
```

---

### 🔍 Exploring & Querying

These commands analyze the build graph *without building*.

#### `bazel query` ⭐

**Analyzes dependencies without building.**

```bash
# What does //app depend on?
bazel query 'deps(//app:main)'

# What targets depend on a library?
bazel query 'rdeps(//:*, //lib:mylib)'

# All test targets in directory
bazel query 'kind(test, //app/...)'

# All cc_library targets
bazel query 'kind(cc_library, //...)'

# Build graph as visualization
bazel query 'deps(//app:main)' --output=graph
```

**Use when:** You want to understand the build structure *without compiling*.

**Common query patterns:**

```bash
deps(X)              # What does X depend on?
rdeps(ROOT, X)       # What depends on X?
direct_deps(X)       # Only direct dependencies
kind(RULE, X)        # Targets of specific type
attr(name, ATTR, X)  # Targets with specific attribute
```

**Real-world examples:**

```bash
# Find all Python binaries
bazel query 'kind(py_binary, //...)'

# Find what depends on OpenSSL (to update when it changes)
bazel query 'rdeps(//:*, @openssl//:crypto)'

# Check for circular dependencies
bazel query 'deps(//foo)'  # Look for cycles in output

# Find all test targets in //src
bazel query 'kind(.*test, //src/...)'
```

---

#### `bazel cquery`

**Query after build analysis (respects build flags).**

```bash
# Query considering specific CPU
bazel cquery 'deps(//app:main)' --cpu=aarch64

# Useful for understanding platform-specific selections
bazel cquery 'deps(//app:main)' --platforms=@platforms//os:linux
```

**Use when:** You need query results that respect build configuration (CPU, platform, etc).

**`query` vs `cquery`:**
- `query` — Faster, before analysis, ignores build flags
- `cquery` — After analysis, respects `--cpu`, `--platforms`, etc

---

### 🔧 Maintenance Commands

These manage the Bazel server and configuration.

#### `bazel clean`

**Removes build outputs.**

```bash
# Quick clean (removes most outputs)
bazel clean

# Deep clean (removes entire output base)
bazel clean --expunge

# Async expunge (background cleanup)
bazel clean --expunge_async
```

**Use when:**
- Build is acting weird
- You want to free disk space
- Switching between incompatible configurations

---

#### `bazel shutdown`

**Stops the Bazel server.**

```bash
bazel shutdown
```

**Use when:**
- You want to free memory
- Server seems stuck
- You're running many isolated builds (scripts)

See [[reference/calling-bazel-from-scripts]] for details on server management.

---

#### `bazel info`

**Displays Bazel configuration.**

```bash
bazel info workspace        # Workspace root
bazel info output_base      # Where outputs go
bazel info bazel-bin        # Build output directory
bazel info server_pid       # Server process ID
bazel info release          # Bazel version
```

**Use when:** You need to understand Bazel's configuration or find output locations.

---

#### `bazel version` / `bazel --version`

**Show version information.**

```bash
bazel version              # Full version info
bazel --version            # Quick version (doesn't start server)
```

---

#### `bazel help`

**Get help for any command.**

```bash
bazel help build           # Help for build command
bazel help --long build    # Detailed help with all options
```

---

## Common Workflows

### Workflow 1: Build → Test → Run

**Typical development cycle:**

```bash
# 1. Write code
vim src/main.cc

# 2. Build to check for compilation errors
bazel build //src:main
# ✅ Compiles successfully

# 3. Run tests
bazel test //src:tests
# ✅ All tests pass

# 4. Run the program
bazel run //src:main -- --config=dev
# ✅ Program works as expected
```

---

### Workflow 2: Explore Dependencies

**Understanding the build structure:**

```bash
# What does my app depend on?
bazel query 'deps(//app:main)'

# Filter to only libraries
bazel query 'kind(cc_library, deps(//app:main))'

# What depends on my library?
bazel query 'rdeps(//:*, //lib:mylib)'

# Find all external dependencies
bazel query 'deps(//app:main)' | grep '@'
```

---

### Workflow 3: Clean and Rebuild

**When something goes wrong:**

```bash
# Try normal clean first
bazel clean
bazel build //...

# If that doesn't help, deep clean
bazel clean --expunge
bazel build //...

# In scripts, ensure isolation
bazel --output_base=/tmp/bazel-$$  build //...
bazel shutdown
```

---

### Workflow 4: Batch Testing

**Run all tests efficiently:**

```bash
# Build everything first (parallelized)
bazel build //...

# Then run all tests
bazel test //...

# Or in one command
bazel test //...

# Run tests by size (small tests first, fast feedback)
bazel test --test_size_filters=small //...
bazel test --test_size_filters=medium,large //...
```

---

## Quick Reference Cheat Sheet

```bash
# Build
bazel build //path:target          # Build single target
bazel build //path/...             # Build all in directory
bazel build -c opt //path:target   # Optimized build

# Test
bazel test //path:target           # Run tests
bazel test --test_filter=Pattern //path  # Run matching tests
bazel test -c dbg //path           # Debug mode

# Run
bazel run //path:target            # Build and run
bazel run //path:target -- ARG1 ARG2  # With arguments

# Query
bazel query 'deps(//path:target)'  # What does it need?
bazel query 'rdeps(//:*, //lib)'   # What needs this lib?
bazel query 'kind(test, //path/...)' # All tests

# Info
bazel info workspace               # Workspace root
bazel info output_base             # Output location

# Maintenance
bazel clean                        # Clean outputs
bazel clean --expunge              # Deep clean
bazel shutdown                     # Stop server
bazel --version                    # Check version
```

---

## Choosing the Right Command

**I want to...**

| Goal | Command | Example |
|------|---------|---------|
| Compile code | `build` | `bazel build //app:main` |
| Run tests | `test` | `bazel test //app:tests` |
| Execute a program | `run` | `bazel run //app:main` |
| Check dependencies | `query` | `bazel query 'deps(//app:main)'` |
| Find what uses a library | `query rdeps` | `bazel query 'rdeps(//:*, //lib)'` |
| Reset everything | `clean --expunge` | `bazel clean --expunge` |
| See where outputs go | `info` | `bazel info bazel-bin` |
| Understand build config | `info` | `bazel info` |
| Stop the server | `shutdown` | `bazel shutdown` |
| Get help | `help` | `bazel help build` |

---

## Pro Tips

### 1. Use `bazel run` for Development
Don't build then manually execute. Use `bazel run`:

```bash
# ❌ Cumbersome
bazel build //app:main
./bazel-bin/app/main --arg

# ✅ Better
bazel run //app:main -- --arg
```

### 2. Use `bazel query` Before Building Large Changes
Understand what you're about to build:

```bash
# Check what depends on the library you're changing
bazel query 'rdeps(//:*, //mylib:lib)'

# See all its dependencies (might be surprising!)
bazel query 'deps(//myapp:app)' | wc -l
```

### 3. Clean Selectively
Don't always use `--expunge`. It's slow:

```bash
# Try regular clean first
bazel clean

# Only use --expunge if necessary
bazel clean --expunge
```

### 4. Filter Tests by Size
Run fast tests first for quicker feedback:

```bash
# Small tests run in seconds
bazel test --test_size_filters=small //...

# Medium tests take longer
bazel test --test_size_filters=medium //...

# Large tests are slow but thorough
bazel test --test_size_filters=large //...
```

---

## See Also

- [[reference/cli-reference]] — Complete command reference with all options
- [[reference/build-options]] — Detailed build and compilation flags
- [[reference/calling-bazel-from-scripts]] — Using Bazel in automation
- [[concepts/fundamentals/targets]] — What targets are
