---
title: "Execution Strategies: How Bazel Runs Build Actions"
category: "reference"
level: "intermediate"
status: "seedling"
sources: []
tags: ["execution", "sandbox", "performance", "hermiticity", "build-strategy"]
related: ["[[concepts/advanced/hermeticity]]", "[[reference/build-options]]", "[[reference/remote-execution]]", "[[troubleshooting/hermeticity-issues]]"]
last_updated: "2026-07-20"
---

# Execution Strategies: How Bazel Runs Build Actions

## What is Sandboxing?

**Sandboxing** is Bazel's technique for isolating build actions in controlled environments. When an action runs in a sandbox, it only has access to:
- Its explicitly declared inputs (source files, dependencies)
- A temporary working directory
- Standard output/error streams

Everything else—system files, environment variables, other projects' artifacts—is hidden or unavailable.

---

## Why Use Sandboxing?

### Problem Without Sandboxing

Imagine you build without sandboxing:

```bash
$ bazel build //my:app
# Action: cc_binary "app" 
# Compiler is /usr/bin/gcc (system installed)
# Compiler can access /usr/include/ (all system headers)
# If you update system libraries, build output might change
```

**Problems:**
- ❌ Different machines with different system libraries → different binaries
- ❌ System updates break builds ("it worked yesterday")
- ❌ Hidden dependencies (compiler accidentally uses /usr/local/ssl)
- ❌ Cache hits ineffective (same source, different binaries across machines)
- ❌ Can't reproduce builds from other machines

### Solution: Sandboxing

```bash
$ bazel build --spawn_strategy=sandboxed //my:app
# Action: cc_binary "app"
# Compiler is provided by rules_cc (declared dependency)
# Compiler can ONLY access declared inputs + temp directory
# System libraries completely hidden
```

**Benefits:**
- ✅ Same source → same binary (on any machine)
- ✅ Catches implicit dependencies (if compiler needs /usr/lib, build fails)
- ✅ Cache reuse across team (same hash → reuse cached binary)
- ✅ Reproducible builds (verifiable)

---

## How Sandboxing Works

### Creating the Sandbox

For each action, Bazel:

1. **Creates temp directory** — `/tmp/bazel-sandbox-12345/`
2. **Populates inputs** — Copies/links only declared inputs into sandbox
3. **Sets restricted environment** — Only whitelisted env vars (PWD, PATH limited)
4. **Runs action** — Executes compiler/linker/tool inside sandbox
5. **Captures outputs** — Collects generated files (`.o`, `.a`, binaries)
6. **Cleans up** — Removes temp directory

```
Before sandbox:
  /usr/include/       (system headers - NOT available)
  /usr/lib/           (system libraries - NOT available)
  /home/user/project/ (source code)
  /home/user/project/src/main.cc

Inside sandbox (/tmp/bazel-sandbox-12345/):
  /tmp/bazel-sandbox-12345/src/main.cc  (linked from source)
  /tmp/bazel-sandbox-12345/compiler/gcc (linked from declared dependency)
  
  Compiler tries to #include <stdio.h>
  → Looks in /usr/include → NOT AVAILABLE in sandbox
  → Build FAILS (catches implicit dependency!)
  
  If you declared @libc//:headers as dependency:
  → Looks in /tmp/bazel-sandbox-12345/libc/include
  → SUCCEEDS (explicit dependency satisfied)
```

### Key Isolation Mechanisms

| Mechanism | Purpose | Example |
|-----------|---------|---------|
| **Filesystem isolation** | Hide system files | `/usr/include` invisible |
| **Environment filtering** | Limit env vars | Only `PATH`, `PWD`, whitelisted vars |
| **Working directory** | Contained output | `/tmp/bazel-sandbox-xyz/` |
| **No network** | Prevent external calls | Can't `curl https://...` |
| **No /tmp access** | Prevent shared state | Can't write to shared `/tmp` |

---

## Execution Strategies Comparison

Bazel supports 5 execution strategies, each with different isolation + performance tradeoffs:

### 1. `sandboxed` (Default on Linux/macOS)

**Isolation: Maximum** | **Speed: Slower**

```bash
bazel build --spawn_strategy=sandboxed //my:app
```

**How it works:**
- Each action runs in isolated sandbox
- Only declared inputs visible
- Perfect for catching implicit dependencies

**Pros:**
- ✅ Maximum hermiticity (catches all undeclared deps)
- ✅ Reproducible (same source → same binary everywhere)
- ✅ Safe for CI/CD (no weird environment surprises)
- ✅ Enables remote caching (deterministic outputs)

**Cons:**
- ❌ Slower (sandbox creation overhead: ~10-50ms per action)
- ❌ Large projects with 10k+ actions can be 20-30% slower
- ❌ Doesn't work on all systems (limited Windows support)

**When to use:**
```bash
# Release builds (correctness > speed)
bazel build -c opt --spawn_strategy=sandboxed //:release

# CI/CD (must be reproducible)
bazel build --spawn_strategy=sandboxed //...

# Hermiticity testing
bazel build --spawn_strategy=sandboxed --sandbox_debug //...
```

**Example: Catching Implicit Dependency**
```starlark
# BUILD.bazel (BROKEN - missing dependency)
cc_binary(
    name = "app",
    srcs = ["main.cc"],
    # Missing: deps = ["@openssl//:crypto"]
)
```

```bash
# Without sandbox: builds successfully (uses /usr/lib/libssl.so)
$ bazel build --spawn_strategy=local //app
# ✅ Success (but non-hermetic!)

# With sandbox: fails (can't find /usr/lib/libssl.so)
$ bazel build --spawn_strategy=sandboxed //app
# ❌ Error: /usr/lib/libssl.so: No such file or directory
# ✅ Caught implicit dependency! Add deps = ["@openssl//:crypto"]
```

---

### 2. `local` (Fast but Less Hermetic)

**Isolation: Minimal** | **Speed: Faster**

```bash
bazel build --spawn_strategy=local //my:app
```

**How it works:**
- Actions run directly on host machine
- Full access to system environment (gcc, /usr/include, all env vars)
- No sandbox creation overhead

**Pros:**
- ✅ Fast (no sandbox overhead, ~10ms faster per action)
- ✅ Works everywhere (Windows, Linux, macOS)
- ✅ Good for interactive development
- ✅ Faster iteration cycles

**Cons:**
- ❌ Not hermetic (can hide implicit system dependencies)
- ❌ System state affects build (different machine = different results)
- ❌ Cache misses if system changes (e.g., gcc updated)
- ❌ No guarantee of reproducibility

**When to use:**
```bash
# Local development (speed is priority)
bazel build --spawn_strategy=local //my:app

# Quick iteration (rebuilding same target)
bazel build --spawn_strategy=local --keep_state //...

# Interactive debugging
bazel build -s --spawn_strategy=local //app  # See actual commands
```

**Trade-off Example:**
```bash
# Machine A (gcc 11 installed)
$ bazel build --spawn_strategy=local //app
# Produces binary_A (compiled with gcc 11)

# Machine B (gcc 12 installed)  
$ bazel build --spawn_strategy=local //app
# Produces binary_B (compiled with gcc 12)
# Cache doesn't help (different system → different binary)

# With sandbox on both:
$ bazel build --spawn_strategy=sandboxed //app
# Both produce binary_C (same compiler version, hermetic)
# Cache helps on both machines!
```

---

### 3. `docker` (Controlled Environment)

**Isolation: Maximum** | **Speed: Slower**

```bash
bazel build --spawn_strategy=docker //my:app
```

**How it works:**
- Each action runs in minimal Docker container
- Container image contains only necessary tools
- Sandbox + container isolation

**Pros:**
- ✅ Maximum hermiticity (container limits access even more than sandbox)
- ✅ Cross-machine consistency (same Docker image → same environment)
- ✅ Catches system-level dependencies
- ✅ Good for CI/CD verification

**Cons:**
- ❌ Requires Docker installed
- ❌ Slower than sandboxed (Docker overhead)
- ❌ More complex setup
- ❌ Not all platforms support Docker containers

**When to use:**
```bash
# Testing for RBE compatibility
$ bazel build --spawn_strategy=docker //...  # Simulates RBE isolation

# Verifying hermiticity before remote caching
$ bazel build --spawn_strategy=docker //app

# CI/CD with maximum isolation guarantee
$ bazel build --spawn_strategy=docker //...
```

**Docker vs Sandboxed:**
```bash
# Sandboxed: uses linux-sandbox, lighter
$ bazel build --spawn_strategy=sandboxed //app

# Docker: uses Docker containers, heavier but more guaranteed
$ bazel build --spawn_strategy=docker //app

# Docker better for:
# - Verifying RBE compatibility (before using remote execution)
# - Maximum isolation guarantee
# - Cross-platform reproducibility verification
```

---

### 4. `worker` (Persistent Process)

**Isolation: None** | **Speed: Fastest**

```bash
bazel build --spawn_strategy=worker //my:app
```

**How it works:**
- Reuses same process for multiple actions
- Tool (compiler, etc.) stays loaded in memory
- Processes requests from Bazel in sequence

**Pros:**
- ✅ Fastest (avoids process startup overhead)
- ✅ Reuses in-memory caches (useful for interpreters like Python)
- ✅ Great for incremental builds
- ✅ Supported everywhere

**Cons:**
- ❌ Stateful (previous action's state can affect next action)
- ❌ Not hermetic (state carries between actions)
- ❌ Cache invalidation tricky (same input but different state = different output)
- ❌ Only works if tool supports persistent workers (not all do)

**When to use:**
```bash
# Development with incremental builds
$ bazel build --spawn_strategy=worker //my/lib

# Testing (pytest, unittest with persistent process)
$ bazel test --spawn_strategy=worker //...

# Fast feedback on local changes
$ bazel build --spawn_strategy=worker //my:app
```

**Example: Why Worker Can Be Unsafe**
```
Action 1 (compile main.cc):
- Input: main.cc, temp flags
- Process: gcc starts, sets FLAG_X = 1
- Output: main.o
- Process keeps running

Action 2 (compile helper.cc):
- Input: helper.cc (same compiler)
- Process: gcc still has FLAG_X = 1 (UNEXPECTED!)
- Output: helper.o (compiled with different flags!)

Result: Same source files, same tools, but different outputs
→ NOT REPRODUCIBLE when using persistent workers
```

**Workers with Isolation:**
Some tools support "worker isolation" (reset state between requests):
- ✅ Protobuf compiler (supports worker mode safely)
- ✅ Python (with worker isolation) 
- ⚠️ C++ compiler (not recommended for worker mode without isolation)

---

### 5. `remote` (Distributed Execution)

**Isolation: Maximum** | **Speed: Depends**

```bash
bazel build --spawn_strategy=remote --remote_executor=grpcs://remote-build-farm.com //my:app
```

**How it works:**
- Actions sent to remote machines (build farm)
- Each machine has Docker sandbox or similar isolation
- Results cached and returned

**Pros:**
- ✅ Scales to 100s of parallel workers
- ✅ Massive speedup for large builds (10x-100x)
- ✅ Hermetic and reproducible
- ✅ Shared cache across team

**Cons:**
- ❌ Network latency (round-trip to remote)
- ❌ Requires infrastructure (build farm)
- ❌ Complex setup
- ❌ Cold builds slower than local (network overhead)

**When to use:**
```bash
# CI/CD with large builds
$ bazel build --spawn_strategy=remote //...

# Team sharing cache
$ bazel build --remote_cache=grpcs://cache.company.com //...

# Cross-team builds (always hermetic)
$ bazel build --spawn_strategy=remote //...
```

See [[reference/remote-execution]] for detailed RBE setup.

---

## Sandbox Implementations

When using `--spawn_strategy=sandboxed`, Bazel automatically selects the best available sandbox implementation for your platform.

### `linux-sandbox` (Maximum Isolation)

**Available on:** Linux with user namespaces

**How it works:**
- Uses Linux Namespaces (User, Mount, PID, Network, IPC)
- Makes entire filesystem read-only except sandbox directory
- Prevents reading/writing files outside sandbox
- Isolates process visibility (action can't see other processes)
- Optionally prevents network access

**Benefits:**
- ✅ Strongest isolation (prevents accidental `rm -rf /home`)
- ✅ Network isolation possible
- ✅ PID isolation ensures all processes killed reliably

**Limitations:**
- ❌ Doesn't work in nested scenarios (Docker containers)
- ⚠️ Requires user namespaces enabled (`/proc/sys/kernel/unprivileged_userns_clone = 1`)

### `darwin-sandbox` (macOS Isolation)

**Available on:** macOS

**How it works:**
- Uses Apple's `sandbox-exec` tool
- Similar to linux-sandbox but for macOS
- Restricts file access and process visibility

**Benefits:**
- ✅ Native macOS sandboxing
- ✅ Similar isolation guarantees to linux-sandbox

**Limitations:**
- ❌ Can't run inside already-sandboxed process
- ❌ Not as comprehensive as linux-sandbox

### `processwrapper-sandbox` (Portable)

**Available on:** All POSIX systems (Linux, macOS, FreeBSD)

**How it works:**
- Creates sandbox directory with symlinks to inputs
- Runs action in sandbox directory
- Moves outputs back to execroot
- Cleans up sandbox directory

**Benefits:**
- ✅ Works on any POSIX system (portable)
- ✅ No special OS features required
- ✅ Falls back when linux/darwin sandboxes unavailable

**Limitations:**
- ⚠️ Action can still read files outside sandbox (if it knows absolute paths)
- ⚠️ Can't prevent filesystem modifications outside sandbox

### Automatic Fallback Chain

Bazel automatically tries sandboxes in priority order:

```
Linux with user namespaces?
  ├─ YES → linux-sandbox
  └─ NO → processwrapper-sandbox

macOS?
  ├─ YES → darwin-sandbox
  └─ NO → processwrapper-sandbox

Docker container on Linux?
  └─ Falls back to processwrapper-sandbox (linux-sandbox unavailable)
```

### Nested Sandboxing

**Scenario:** Running Bazel inside Docker or sandbox

**Problem:** linux-sandbox inside Docker fails (nested restrictions)

**Solution:** Bazel automatically uses processwrapper-sandbox (less strict, but works)

**Force specific strategy if needed:**
```bash
# Force processwrapper if nested
bazel build --spawn_strategy=processwrapper-sandbox //...

# Or fall back on error
bazel build --spawn_strategy=linux-sandbox,processwrapper-sandbox //...
```

---

## Downsides to Sandboxing

### Setup/Teardown Overhead

**Cost:** ~10-50ms per action

**Mitigation:**
- `--reuse_sandbox_directories` — Reuse sandbox dirs across builds (faster)
- On Linux: overhead rarely exceeds a few percent

### Tool Cache Disabled

**Problem:** Each sandbox is fresh (no cache in tool)

**Mitigation:**
- Use persistent workers (`--spawn_strategy=worker`)
- Trade-off: weaker sandbox guarantees

### Worker Memory Usage

**Problem:** Multiplex workers require special support for sandboxing

**Impact:** Workers without multiplex support run as singleplex (uses more memory)

---

## Hybrid Strategies

### `dynamic` (Try Both, Use First)

```bash
bazel build --spawn_strategy=dynamic //my:app
```

**How it works:**
- Start action locally (sandboxed)
- Simultaneously send to remote worker
- Use whichever finishes first

**Result:** Best of both worlds (fast local, fallback to remote)

### Fallback Chain

```bash
# Try remote first, fallback to local sandbox
bazel build \
  --spawn_strategy=remote \
  --experimental_fallback_strategy=sandboxed \
  //...
```

---

## Performance Impact

Typical overhead per action by strategy:

| Strategy | Overhead | Total for 1000 actions |
|----------|----------|----------------------|
| `local` | ~0ms | ~0s |
| `worker` | ~1ms | ~1s |
| `sandboxed` | ~10-50ms | ~10-50s |
| `docker` | ~50-200ms | ~50-200s |
| `remote` | ~100-500ms* | ~100-500s* |

*Remote includes network latency; offsets by massive parallelism

### Real Project Examples

**Small project (100 actions):**
```
Strategy         Time
─────────────────────
local           30s  ✅ Fastest
worker          35s
sandboxed       40s
remote          120s (overhead dominates)
```

**Large project (10,000 actions):**
```
Strategy         Time (parallel)
─────────────────────────────
local           300s (parallelism capped by cores)
worker          280s
sandboxed       350s
remote          60s  ✅ Fastest (100+ workers)
```

**Insight:** Sandboxing overhead matters for small projects, but remote execution's parallelism wins for large monorepos.

---

## Configuring Strategies

### Global Default (in .bazelrc)

```
# .bazelrc
build --spawn_strategy=sandboxed    # Default: hermetic
test --spawn_strategy=sandboxed     # Tests must be hermetic
run --spawn_strategy=local          # Quick iteration
```

### Per-Build Override

```bash
# Override default
bazel build --spawn_strategy=local //my:app

# Use multiple strategies (Bazel picks first available)
bazel build --spawn_strategy=remote --spawn_strategy=sandboxed //...
```

### Per-Action Override

```starlark
# BUILD.bazel
cc_library(
    name = "my_lib",
    srcs = ["lib.cc"],
    exec_properties = {
        "supports-workers": "1",  # This action supports worker mode
    },
)

# Bazel can choose worker strategy for this action
```

### Environment Variable

```bash
# Set default strategy
export BAZEL_BUILD_OPTS="--spawn_strategy=sandboxed"
bazel build //...
```

---

## Debugging Sandbox Issues

### See Sandbox Failures

```bash
# Keep sandbox dir for inspection
bazel build --sandbox_debug //my:app

# If build fails, sandbox stays at:
# /tmp/bazel-sandbox-*/
# Inspect what files were available
```

### Test Sandboxing

```bash
# First: build with local (fast)
$ bazel build --spawn_strategy=local //app
# Success

# Then: build with sandbox (strict)
$ bazel build --spawn_strategy=sandboxed //app
# If this FAILS, you have undeclared dependencies!
```

### Inspect Action Inputs

```bash
# Show what inputs each action received
bazel aquery 'attr(name, "my_target", //...)'
```

---

## Best Practices

### Development

```bash
# Fast iteration: use local strategy
bazel build --spawn_strategy=local //my:app

# But periodically verify hermeticity:
bazel build --spawn_strategy=sandboxed //...  # Should still pass
```

### Testing

```bash
# Tests must pass with sandboxed (catches implicit deps)
bazel test --spawn_strategy=sandboxed //...
```

### CI/CD

```bash
# Release builds: maximum reproducibility
bazel build -c opt --spawn_strategy=sandboxed //...

# Or use remote for parallelism
bazel build -c opt --spawn_strategy=remote //...
```

### RBE-Safe Code

```bash
# Before using remote execution:
# 1. Test locally with docker sandbox
bazel build --spawn_strategy=docker //...

# 2. Verify no system dependencies
bazel build -s --spawn_strategy=sandboxed //... 2>&1 | grep /usr/

# 3. Then use remote execution confidently
bazel build --spawn_strategy=remote //...
```

---

## Troubleshooting

### "Build passes with --spawn_strategy=local but fails with sandboxed"

**Problem:** Undeclared dependencies (implicit system access)

**Solution:**
1. Identify missing dependency from sandbox error
2. Add to `deps` in BUILD.bazel
3. Verify build passes with `--spawn_strategy=sandboxed`

```starlark
# BEFORE (broken)
cc_binary(
    name = "app",
    srcs = ["main.cc"],
)

# AFTER (fixed)
cc_binary(
    name = "app",
    srcs = ["main.cc"],
    deps = ["@openssl//:crypto"],  # Added missing dep
)
```

### "Sandbox fails on my machine but works elsewhere"

**Problem:** Your machine has unusual environment

**Solution:**
1. Test with Docker sandbox (controlled environment)
2. Or run remote execution (isolated workers)

### "Sandboxed builds are too slow"

**Problem:** Overhead of sandbox creation

**Solutions:**
1. Use `--spawn_strategy=dynamic` (local + remote fallback)
2. Use remote execution for parallelism
3. Accept overhead if hermiticity is critical

---

## See Also

- [[concepts/advanced/hermeticity]] — Why reproducibility matters
- [[reference/build-options]] — Compilation modes and other flags
- [[reference/remote-execution]] — Remote execution setup
- [[troubleshooting/hermeticity-issues]] — Fixing non-hermetic builds
