---
title: "Debugging Hermeticity Issues"
category: "troubleshooting"
level: "intermediate"
status: "growing"
sources: ["Hermeticity    Bazel.md"]
tags: ["hermeticity", "debugging", "reproducibility", "sandboxing"]
graph-group: "troubleshooting"
related: ["[[concepts/advanced/hermeticity]]", "[[troubleshooting/build-failures]]"]
last_updated: "2026-07-19"
---

# Debugging Hermeticity Issues

A hermetic build produces identical outputs from identical inputs. Non-hermetic builds leak information from the host machine, causing non-deterministic behavior.

## Identifying Non-Hermetic Builds

### Test 1: Null Sequential Build

Run the same build twice without changing inputs—the second build should rebuild nothing.

```bash
# First build
$ bazel build //:app
# INFO: 3 processes: 3 linux-sandbox.

# Second build (should rebuild nothing)
$ bazel build //:app
# INFO: 0 processes (no rebuilds)  ✓ Hermetic
```

**If rebuilding occurs:** Your build is non-hermetic.

### Test 2: Cross-System Build Verification

Build on two different machines and compare output hashes:

```bash
# Machine A (Linux x86_64)
$ bazel build //:app
$ sha256sum bazel-bin/app > hash_a.txt

# Machine B (Linux x86_64, same code and config)
$ bazel build //:app
$ sha256sum bazel-bin/app > hash_b.txt

# Compare hashes
$ diff hash_a.txt hash_b.txt
# If different: build is non-hermetic
```

### Test 3: Docker Sandbox

Run builds in a minimal container with only source code and explicit tools:

```dockerfile
FROM alpine:latest

# Copy ONLY source code—no system tools
COPY . /workspace

WORKDIR /workspace

# Try to build; if it fails, you have implicit system dependencies
RUN bazel build //...
```

If the build fails in the container but succeeds on your machine, you're relying on host system tools.

## Common Non-Hermeticity Sources

### 1. Timestamps and Determinism

**Problem:** Actions include build timestamps or sequence numbers.

```starlark
# BAD: Includes current timestamp (non-deterministic)
genrule(
    name = "version",
    cmd = "date > $@",
)

# BAD: Sequential number (non-deterministic)
genrule(
    name = "build_id",
    cmd = "echo $(date +%s) > $@",
)
```

**Fix:** Hardcode deterministic values.

```starlark
# GOOD: Deterministic version
genrule(
    name = "version",
    cmd = "echo '1.0.0' > $@",
)

# GOOD: Use git commit hash (deterministic)
genrule(
    name = "build_id",
    srcs = [".git/HEAD"],
    cmd = "cat $(location .git/HEAD) > $@",
)
```

### 2. System Binaries

**Problem:** Using `/usr/bin` tools or assuming system versions.

```starlark
# BAD: Relies on /usr/bin/python (version varies by system)
genrule(
    name = "process_data",
    cmd = "/usr/bin/python3 process.py > $@",
)

# BAD: Assumes system C++ compiler
cc_toolchain(
    compiler = "/usr/bin/c++",  # Version differs across systems
)
```

**Fix:** Download specific tool versions.

```starlark
# GOOD: Declare Python as a dependency
genrule(
    name = "process_data",
    tools = ["@python3_3_11//:bin/python"],
    cmd = "$(location @python3_3_11//:bin/python) process.py > $@",
)

# GOOD: Use versioned toolchain
cc_toolchain(
    compiler = "@gcc_11_2//:bin/gcc",  # Specific version
)
```

### 3. Absolute Paths

**Problem:** Hardcoding absolute paths that vary across systems.

```starlark
# BAD: Absolute paths (differs across machines)
cc_binary(
    name = "app",
    linker_flags = ["-L/usr/local/lib"],
)

# BAD: User-specific paths
genrule(
    cmd = "gcc -o $@ -I/home/alice/include $<",
)
```

**Fix:** Use relative paths and explicit dependencies.

```starlark
# GOOD: Use relative paths and rules
cc_binary(
    name = "app",
    deps = [
        "@openssl//:ssl_lib",  # Bazel-managed dependency
    ],
)

# GOOD: Use toolchain from rules
genrule(
    name = "compile",
    tools = ["@gcc//:bin/gcc"],
    cmd = "$(location @gcc//:bin/gcc) -o $@ $<",
)
```

### 4. Writing to Source Tree

**Problem:** Build modifies source files, breaking subsequent builds of different targets.

```starlark
# BAD: Writes to source tree
genrule(
    name = "generate",
    cmd = "python generate.py",  # Writes to src/ directory
    outs = [],  # No outputs declared
)
```

**Fix:** Generate outputs to `bazel-bin/` and declare them.

```starlark
# GOOD: Outputs go to bazel-bin/
genrule(
    name = "generate",
    cmd = "python $(location :generator) > $@",
    outs = ["generated.cc"],  # Declared outputs
    tools = [":generator"],
)
```

### 5. Non-Deterministic Compression/Serialization

**Problem:** Tools that produce different outputs from same inputs (random IDs, dict ordering, etc.).

```starlark
# BAD: Zip files may have different timestamps or ordering
genrule(
    cmd = "zip -r $@ src/",  # May not be deterministic
)

# BAD: JSON serialization (dict ordering)
genrule(
    cmd = "python -c 'import json; json.dump(data, open(\"$@\", \"w\"))'",
)
```

**Fix:** Use deterministic tools and options.

```starlark
# GOOD: Use deterministic zip
genrule(
    cmd = "zip -r -D $@ src/",  # -D disables timestamps
)

# GOOD: Sort JSON keys
genrule(
    cmd = "python -c 'import json; json.dump(data, open(\"$@\", \"w\"), sort_keys=True)'",
)
```

### 6. Workspace Rules with Side Effects

**Problem:** Workspace rules that download files or run scripts non-deterministically.

```starlark
# BAD: Downloads from URL without hash verification
http_archive(
    name = "deps",
    urls = ["https://example.com/archive.tar.gz"],
    # No sha256 verification
)

# BAD: Runs arbitrary script during workspace setup
new_local_repository(
    name = "tools",
    path = "/opt/tools",  # Path varies across systems
)
```

**Fix:** Use hash verification and explicit dependencies.

```starlark
# GOOD: Hash-verified download
http_archive(
    name = "deps",
    urls = ["https://example.com/archive.tar.gz"],
    sha256 = "abc123...",  # Ensures exact version
    strip_prefix = "archive",
)

# GOOD: Bazel-managed dependencies
bazel_dep(name = "tools", version = "1.2.3")
```

## Fixing Non-Hermetic Builds

### Step 1: Enable Sandboxing

Bazel can sandbox actions to catch implicit dependencies.

```bash
# Enable sandboxing (limits action's file access)
bazel build --sandbox_debug //:app

# If build fails in sandbox but succeeds without it, you have
# implicit system dependencies
```

### Step 2: Log Workspace Rules

Find non-deterministic workspace rule execution:

```bash
bazel build --experimental_workspace_rules_log_file=/tmp/ws.log //:app

# Review /tmp/ws.log for actions that might be non-hermetic
```

### Step 3: Enable Strict Sandboxing

For per-action sandboxing (prevents all implicit system access):

```bash
bazel build --spawn_strategy=sandboxed //:app

# Each action runs in isolated sandbox; only declared inputs visible
```

### Step 4: Reproducibility Testing

Build the same target multiple times and verify identical outputs:

```bash
# Build 3 times, capture hashes
for i in {1..3}; do
  bazel clean
  bazel build //:app
  sha256sum bazel-bin/app >> hashes.txt
done

# All hashes should be identical
sort hashes.txt | uniq
# If fewer than 3 unique hashes: non-hermetic
```

### Step 5: System-Wide Testing

```bash
# Test on clean VM or Docker container
docker run -it ubuntu:latest bash
cd /workspace
bazel build //...  # Should work without system dependencies
```

## Checking Specific Action Execution

### Inspect Action Command

```bash
# See exact commands Bazel runs
bazel build -s //:app 2>&1 | grep -A5 "Linking"
```

### Check Action Inputs

```bash
# List all files an action declared as inputs
bazel aquery 'outputs("bazel-bin/app")' --output=jsonproto 2>&1 | grep -A10 inputs
```

## Best Practices

1. **Test locally first** — Run null sequential builds before pushing to CI
2. **Use remote execution** — Remote cache exposes hermeticity issues (different machines)
3. **Explicit dependencies** — If an action needs something, declare it
4. **Version tools** — Pin compiler, Python, and other tool versions
5. **Avoid system assumptions** — Don't assume `/usr/bin/gcc` exists
6. **Document toolchains** — Explain toolchain versions and origins
7. **CI verification** — Run builds on multiple OS/CPU combinations to catch non-hermeticity

## Debugging Tools

| Tool | Purpose |
|------|---------|
| `bazel build -s` | Show command lines (find bare `gcc`, `/usr/bin` references) |
| `bazel aquery` | Inspect action inputs/outputs |
| `--sandbox_debug` | Keep sandbox failures visible for inspection |
| `--experimental_workspace_rules_log_file` | Log workspace rule execution |
| `--spawn_strategy=sandboxed` | Force per-action sandboxing |
| Docker | Test in minimal environment |
| `sha256sum` | Verify output reproducibility |

---

**Key Insight:** Hermeticity is crucial for reliable, fast, and reproducible builds. Invest in finding and fixing non-hermeticity early.
