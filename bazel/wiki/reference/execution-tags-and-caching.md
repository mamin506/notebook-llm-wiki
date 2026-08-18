---
title: "Execution Tags and Caching Control"
category: "reference"
level: "intermediate"
status: "growing"
sources: ["Common definitions.md", "Remote Caching.md", "Command-Line Reference.md"]
tags: ["#tags", "#execution", "#caching", "#remote-execution", "#local-only", "#precedence"]
related: ["[[reference/execution-strategies]]", "[[reference/remote-execution]]", "[[concepts/advanced/hermeticity]]", "[[reference/build-command-options]]"]
last_updated: "2026-07-29"
---

# Execution Tags and Caching Control

Control where and how Bazel executes actions using tags. This is critical for non-portable build artifacts like kernel drivers, hardware-specific binaries, or system-dependent outputs.

---

## Overview

Bazel modifies behavior when it finds specific keywords in the `tags` attribute of rules or the `execution_requirements` of Starlark actions. These tags control:

- Whether an action runs locally or remotely
- Whether outputs are cached locally or remotely
- Whether sandboxing is used
- Whether network access is allowed

---

## Execution Tags Reference

### `local`

**Effect:** Forces the action to run locally, without sandboxing. Disables remote caching, remote execution, and sandboxing.

**Use case:** Actions that depend on local machine state, require direct access to system resources, or produce non-portable outputs.

**Example:**
```python
genrule(
    name = "process",
    tags = ["local"],
    cmd = "local_tool $(SRCS) > $@",
)
```

**Equivalent to:**
- Setting `local = True` attribute on test or genrule (deprecated)
- Using `no-remote`, `no-cache`, and `no-sandbox` together

---

### `no-remote`

**Effect:** Prevents the action from being:
- Remotely executed (but not remote caching)
- Cached remotely
- Same as combining `no-remote-exec` + `no-remote-cache`

**Use case:** Sensitive builds, non-portable outputs, or when you want local caching but not remote.

**Example:**
```python
cc_binary(
    name = "app",
    tags = ["no-remote"],
    srcs = ["main.cc"],
)
```

**Impact:**
```
✅ Can use local disk cache
❌ Cannot use remote cache
❌ Cannot execute remotely
✅ Can be sandboxed locally
```

---

### `no-remote-exec`

**Effect:** Prevents remote execution but allows remote caching.

**Use case:** When outputs are portable but you want to guarantee execution happens locally for auditing or compliance reasons.

**Example:**
```python
py_binary(
    name = "config_generator",
    tags = ["no-remote-exec"],
    srcs = ["generate.py"],
)
```

**Impact:**
```
✅ Can use local cache
✅ Can use remote cache (read/write)
❌ Cannot execute remotely
✅ Can be sandboxed
```

---

### `no-remote-cache`

**Effect:** Prevents remote caching but allows remote execution.

**Use case:** Rarely used; when you want parallelism from remote execution but outputs shouldn't be shared.

**Example:**
```python
# Not typical, but possible
genrule(
    name = "nondeterministic_tool",
    tags = ["no-remote-cache"],
    cmd = "random_output > $@",
)
```

**Impact:**
```
✅ Can use local cache
❌ Cannot use remote cache
✅ Can execute remotely
✅ Can be sandboxed
```

---

### `no-cache`

**Effect:** Prevents both local and remote caching. Action runs every time.

**Use case:** Actions that depend on external state or must run unconditionally (timestamps, current time, etc.).

**Example:**
```python
genrule(
    name = "build_info",
    tags = ["no-cache"],
    cmd = "echo 'Built at $(date)' > $@",
)
```

**Warning:** Use `no-cache` instead of relying on timestamps or random data, which breaks hermeticity.

**Impact:**
```
❌ Cannot use local cache
❌ Cannot use remote cache
✅ Can execute remotely (but wasteful)
```

---

### `no-sandbox`

**Effect:** Runs the action without sandboxing. Action can access the local filesystem, environment variables, and system state.

**Use case:** Actions that legitimately need host system access (system library checks, hardware detection).

**Warning:** Breaking sandboxing hides dependencies and breaks hermeticity. Use only when necessary.

**Example:**
```python
genrule(
    name = "detect_hardware",
    tags = ["no-sandbox"],
    cmd = "check_cpu_flags > $@",
)
```

**Impact:**
```
✅ Can use local cache
✅ Can use remote cache
❌ Cannot be sandboxed
✅ Can execute remotely (but defeats purpose)
```

---

### `no-remote-cache-upload`

**Effect:** Prevents uploading outputs to remote cache, but allows reading from it.

**Use case:** CI builds that shouldn't pollute the shared cache (experimental builds, temporary branches).

**Example:**
```python
cc_binary(
    name = "experimental",
    tags = ["no-remote-cache-upload"],
    srcs = ["experimental.cc"],
)
```

**Impact:**
```
✅ Can use local cache
✅ Can read from remote cache
❌ Cannot upload to remote cache
```

---

### `requires-network`

**Effect:** Allows network access from inside the sandbox. By default, sandboxed actions cannot access the network.

**Use case:** Actions that need to download files or connect to external services (only when unavoidable).

**Example:**
```python
genrule(
    name = "fetch_data",
    tags = ["requires-network"],
    cmd = "curl https://example.com/data > $@",
)
```

**Warning:** Network access breaks hermeticity and reproducibility. Avoid when possible; prefer pre-downloaded dependencies.

---

### `block-network`

**Effect:** Explicitly blocks network access inside the sandbox (default behavior, rarely needed).

**Use case:** Documenting that an action must not access the network.

**Example:**
```python
py_test(
    name = "offline_test",
    tags = ["block-network"],
    srcs = ["test.py"],
)
```

---

### `requires-fakeroot`

**Effect:** Runs the action as uid/gid 0 (root). Linux only.

**Use case:** Actions that need to create files with specific ownership or permissions.

**Example:**
```python
sh_test(
    name = "permission_test",
    tags = ["requires-fakeroot"],
    srcs = ["test.sh"],
)
```

---

## Test-Specific Tags

### `exclusive`

**Effect:** Forces test to run alone, sequentially, after all other tests and builds complete.

**Use case:** Tests that interfere with others or require exclusive system access.

```python
cc_test(
    name = "system_test",
    tags = ["exclusive"],
    srcs = ["system_test.cc"],
)
```

---

### `exclusive-if-local`

**Effect:** Runs exclusively if executed locally; runs in parallel if executed remotely.

**Use case:** Tests that conflict with other local tests but can run safely on remote workers.

```python
py_test(
    name = "database_test",
    tags = ["exclusive-if-local"],
    srcs = ["db_test.py"],
)
```

---

### `manual`

**Effect:** Excludes the target from wildcard expansion (`...`, `:*`, `:all`).

**Use case:** Tests that require specific setup or flags.

```python
cc_test(
    name = "integration_test",
    tags = ["manual"],
    srcs = ["integration_test.cc"],
)
```

**Build normally with:** `bazel test //:integration_test`

**Not included in:** `bazel test //...`

---

### `external`

**Effect:** Forces unconditional execution regardless of `--cache_test_results`.

**Use case:** Tests that should always run (flakiness detection, external service checks).

```python
sh_test(
    name = "health_check",
    tags = ["external"],
    srcs = ["check.sh"],
)
```

---

## Real-World Examples

### Kernel Driver (Non-Portable Binary)

```python
kernel_module(
    name = "mydriver",
    srcs = ["driver.c"],
    tags = ["local", "no-remote"],  # Run locally, don't cache remotely
)
```

**Why:**
- Kernel modules only work on the kernel version they were compiled for
- .ko files are not portable across machines
- `local` ensures it runs where it's needed
- `no-remote` prevents sharing via remote cache

---

### Configuration Generator (Must Run Locally)

```python
genrule(
    name = "generate_config",
    outs = ["config.json"],
    cmd = "generate_config.sh $(machine_type) > $@",
    tags = ["no-remote-exec"],  # Run locally, but OK to cache
)
```

**Why:**
- Configuration depends on local machine state
- Output is still deterministic and shareable
- `no-remote-exec` guarantees local execution for auditing
- Remote cache is OK

---

### CI Experimental Build

```python
cc_binary(
    name = "experimental_app",
    srcs = ["experiment.cc"],
    tags = ["no-remote-cache-upload"],  # Don't pollute shared cache
)
```

**Why:**
- Experimental features shouldn't affect team cache
- Can still benefit from cache hits
- Doesn't upload results back

---

### Hardware Detection

```python
genrule(
    name = "detect_simd",
    outs = ["simd_config.h"],
    cmd = "detect_simd.sh > $@",
    tags = ["no-sandbox", "no-remote"],  # Needs hardware access
)
```

**Why:**
- Must run on actual hardware to detect SIMD support
- Sandbox hides CPU capabilities
- Can't run remotely (different hardware)

---

## Starlark Actions

In custom rules, use `execution_requirements` dictionary:

```python
def my_rule_impl(ctx):
    output = ctx.actions.declare_file(ctx.attr.name)
    
    ctx.actions.run_shell(
        command = "...",
        outputs = [output],
        execution_requirements = {
            "local": "1",           # Same as tag
            "no-remote": "1",
            "requires-network": "1",
            "cpu:4",                # Needs 4 CPU cores
            "memory:2GB",           # Needs 2GB RAM
            "custom-key": "value",  # Custom keys for RBE
        },
    )
```

---

## How Tags Work: The Propagation Mechanism

### Tags → Execution Requirements Conversion

Bazel **converts `tags` into `execution_requirements` internally**:

```
Step 1: User adds tags in BUILD file
  kernel_module(
      name = "mydriver",
      tags = ["local", "no-remote"],
  )

Step 2: Bazel propagates tags to execution_requirements
  (controlled by --incompatible_allow_tags_propagation, default: true)

Step 3: Actions execute based on merged execution_requirements
  execution_requirements = {
      "local": "1",           # from tags
      "no-remote": "1",       # from tags
      ... + any set by rule impl
  }
```

**Control this with:** `--[no]incompatible_allow_tags_propagation` (default: true)

If set to `false`, tags are NOT propagated to execution_requirements (rarely needed).

---

## Precedence Rules

When rule author's `execution_requirements` and rule user's `tags` conflict:

```
Priority (highest to lowest):
  1. Rule implementation's execution_requirements
     (hardcoded in Starlark; NOT overridable by tags or CLI)
  
  2. Tags converted to execution_requirements
     (BUILD-file tags; can be added by user)
  
  3. Command-line flags (--spawn_strategy, etc)
     (lowest priority; can be overridden by tags)

Example 1: Rule impl sets hard requirement
  Rule implementation: execution_requirements = {"local": "1"}
  Build file: tags = ["no-remote"]           ← extra constraint (OK)
  Command: bazel build --spawn_strategy=remote
  Result: ✅ Action is local (impl requirement wins)

Example 2: Tags complement rule impl
  Rule implementation: (no execution_requirements)
  Build file: tags = ["no-remote"]
  Result: ✅ Action uses "no-remote" requirement

Example 3: Try to override impl (fails silently)
  Rule implementation: execution_requirements = {"local": "1"}
  Build file: tags = []
  Command: bazel build --spawn_strategy=remote
  Result: ⚠️ Command line flag is ignored; action still runs locally
```

**Key principle:** Rule author's `execution_requirements` is the **final authority**—tags and CLI flags cannot remove it, only add to it.

This makes sense: the rule author knows their rule's requirements (e.g., "kernel drivers MUST be local"), and those requirements can't be violated by users or build flags.

---

## Combining Tags

**Safe combinations:**

```python
tags = ["no-remote", "no-cache"]          # ✅ Always local, never cached
tags = ["local"]                           # ✅ Implicit no-remote + no-cache
tags = ["no-remote-exec"]                  # ✅ Local execution, remote cache OK
tags = ["no-sandbox", "local"]             # ✅ No sandbox, local only
```

**Redundant combinations:**

```python
tags = ["local", "no-remote"]              # ✅ OK but "local" is sufficient
tags = ["no-remote-cache", "no-remote-exec"]  # ✅ OK but "no-remote" is shorter
```

---

## Performance Implications

| Tag | Performance | Caching | Parallelism |
|-----|-------------|---------|------------|
| `local` | ⚠️ Slowest (local only) | Local only | No parallelism |
| `no-remote` | ⚠️ Local + local cache | Local + local cache | Limited to local cores |
| `no-remote-exec` | ⚠️ Local execution (cache OK) | Both caches | Limited parallelism |
| `no-cache` | ❌ Slowest (no caching) | Never cached | Can parallelize but wasteful |
| Default | ✅ Fastest (can use RBE) | Both + remote | Maximum parallelism |

---

## See Also

- [[reference/execution-strategies]] — How actions are executed (sandboxed, local, remote)
- [[reference/remote-execution]] — Setting up remote execution infrastructure
- [[concepts/advanced/hermeticity]] — Why hermetic builds matter
