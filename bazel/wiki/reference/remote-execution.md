---
title: "Remote Execution and Build Caching"
category: "reference"
level: "intermediate"
status: "seedling"
sources: ["Remote Execution Overview.md", "Adapting Bazel Rules for Remote Execution.md"]
tags: ["#remote-execution", "#caching", "#rbe", "#performance"]
related: ["[[concepts/advanced/hermeticity]]", "[[patterns/dependency-management]]", "[[troubleshooting/performance]]"]
last_updated: "2026-07-19"
graph-group: "reference"
---

# Remote Execution and Build Caching

Remote execution (RBE) allows Bazel to distribute build and test actions across multiple machines, enabling faster builds, consistent environments, and result caching across your team.

## Overview

By default, Bazel executes builds and tests on your local machine. Remote execution uses an open-source **gRPC protocol** (part of Bazel's Remote Execution APIs) to coordinate execution across a cluster of workers.

### Benefits

1. **Faster Builds** — Parallelize actions across many machines instead of being limited by local CPU cores
2. **Consistent Environment** — All developers and CI build against the same hermetic, defined environment
3. **Shared Cache** — Build outputs are cached centrally; one build's results are available to all developers (dramatically reduces rebuild time for unchanged code)
4. **Reproducibility** — Same inputs always produce same outputs (via hermeticity)

### Trade-offs

- **Network Overhead** — For small, fast builds, local execution may be faster than network latency for RBE
- **Setup Complexity** — Requires infrastructure and configuration
- **Hermiticity Enforcement** — Build rules must be truly hermetic (no system dependencies, no timestamps, etc.)

## Requirements for Remote Execution

Remote execution imposes strict requirements on the build for reproducibility and safety:

### 1. Hermeticity

All build actions must be **hermetic** — same inputs always produce identical outputs.

**Requirements:**
- No system-specific files (e.g., no absolute paths, no reading from `/usr/include`)
- No non-deterministic operations (timestamps, UUIDs, random numbers)
- All dependencies must be declared and available in the build sandbox
- No access to environment variables (unless explicitly whitelisted)

**Example of non-hermetic rule:**
```python
genrule(
    name = "bad_rule",
    cmd = "echo $(date) > $@",  # BAD: non-deterministic output
)
```

**Hermetic version:**
```python
genrule(
    name = "good_rule",
    cmd = "echo 'constant value' > $@",  # Good: deterministic
)
```

### 2. Sandbox Isolation

Actions run in isolated sandboxes with:
- Only declared inputs available
- Restricted environment variables
- No network access (by default)
- No access to the local filesystem

**Implication:** Every tool, library, and file needed must be declared in `srcs`, `deps`, `data`, or `tools`.

### 3. Self-Contained Toolchains

All compilation tools must be shipped as dependencies (not assumed to exist on the remote machine).

**Example:**
```python
cc_binary(
    name = "app",
    srcs = ["main.cc"],
    toolchain = "@bazel_tools//tools/cc_toolchain",  # Explicitly declared
)
```

### 4. Reproducibility

Build outputs must be deterministic:
- No timestamps embedded in binaries (unless explicitly stamped)
- No version information from system
- All tool versions fixed (not "latest")
- Link order must be deterministic

## Configuring Remote Execution

### 1. Set Execution Strategy

Tell Bazel to use remote execution:

**.bazelrc:**
```
# Use remote execution for all actions
build --spawn_strategy=remote
test --spawn_strategy=remote

# Fallback to local if remote fails
build --strategy=Genrule=local

# Or use a hybrid approach:
build --spawn_strategy=dynamic
```

**Strategies:**
- `local` — Execute on local machine
- `remote` — Execute on remote worker
- `dynamic` — Try both, use whichever finishes first (hybrid)
- `worker` — Persistent worker process (fast for many small actions)
- `sandbox` — Sandboxed local execution (ensures hermeticity)

### 2. Configure Remote Server

Specify the Remote Execution server:

**.bazelrc:**
```
# Google Cloud Build
build --remote_executor=grpcs://remotebuildexecution.googleapis.com

# Self-hosted RBE (e.g., BuildBarn)
build --remote_executor=grpcs://your-rbe-server.example.com:443

# Remote cache only (no RBE, just cache):
build --remote_cache=grpcs://your-cache-server:443
```

### 3. Configure Remote Cache

Separate from execution, you can cache build results:

**.bazelrc:**
```
# Local disk cache
build --disk_cache=~/.bazel_cache

# Remote cache (result caching only, no execution)
build --remote_cache=grpcs://your-cache.example.com

# Authentication (for private caches)
build --google_credentials=/path/to/credentials.json
```

## Adapting Rules for Remote Execution

### Hermetic Compilation

Ensure all files and tools are declared:

```python
cc_library(
    name = "mylib",
    srcs = ["mylib.cc"],
    hdrs = ["mylib.h"],
    deps = [
        # All dependencies must be declared
        "@openssl//:ssl",
        "@zlib//:z",
    ],
    # Don't assume system includes like /usr/include
    copts = ["-Iinclude"],  # Relative to package
    linkopts = ["-lm"],     # System library (allowed)
)
```

### Avoiding Non-Determinism

**Bad (non-deterministic):**
```python
genrule(
    name = "timestamp",
    cmd = "date > $@",
)
```

**Good (deterministic):**
```python
genrule(
    name = "version",
    srcs = [":version.txt"],
    outs = ["version.h"],
    cmd = "cat $(location :version.txt) > $@",
)
```

### Stamping (Build Metadata)

If you need version/commit info, use Bazel's stamping:

```python
cc_binary(
    name = "app",
    srcs = ["main.cc"],
    stamp = 1,  # Embed build info
    linkstamp = ":linkstamp",  # Link-time version info
)
```

Access via Bazel's built-in build info.

### Platform Configuration

Use platform/toolchain API (not legacy `--cpu` flags):

```python
cc_binary(
    name = "app",
    srcs = ["main.cc"],
    # Declare target platform constraints
)

# Build with:
# bazel build //:app --platforms=@platforms//os:linux --platforms=@platforms//cpu:x86_64
```

## Remote Caching Strategy

### Three-Tier Caching

1. **Local Disk Cache** — Fast, local, survives system restart
   ```
   build --disk_cache=~/.bazel_cache
   ```

2. **Team/CI Cache** — Shared across developers and CI
   ```
   build --remote_cache=grpcs://cache.company.example.com
   ```

3. **Build Farm (RBE)** — Distributed execution
   ```
   build --remote_executor=grpcs://rbe.company.example.com
   ```

### Cache Invalidation

Caching is **content-addressed** — inputs hash to outputs. If inputs change, cache automatically misses:

- Source code change → new hash → cache miss → rebuild
- Tool version change → new hash → cache miss → rebuild
- Dependency version change → new hash → cache miss → rebuild

This is why hermeticity is critical — non-hermetic builds can pollute the cache with incorrect results.

## Common RBE Services

- **Google Cloud Build** (managed, GCP-integrated)
- **BuildBarn** (open-source, self-hosted)
- **Buildfarm** (Apache-licensed, open-source)
- **Goma** (distributed compilation service)

## Troubleshooting RBE

**Issue: "Action failed: failed to upload file"**
- Check network connectivity to RBE server
- Verify credentials/authentication

**Issue: "Hermiticity violation: system file /usr/include/..."**
- Don't assume system libraries; declare them as dependencies
- Use `--incompatible_strict_action_env` to catch violations

**Issue: "Cache miss on identical code"**
- Check that all dependencies have fixed versions (not "latest")
- Ensure build is truly deterministic (no timestamps, UUIDs)

**Issue: "RBE slower than local build"**
- RBE has network overhead; better for large builds than small ones
- Consider `--spawn_strategy=dynamic` for hybrid execution

## See Also

- [[concepts/advanced/hermeticity]] — Reproducible builds
- [[reference/build-options]] — Build options and flags
- [[patterns/dependency-management]] — Managing external dependencies
- [[troubleshooting/performance]] — Build performance optimization
