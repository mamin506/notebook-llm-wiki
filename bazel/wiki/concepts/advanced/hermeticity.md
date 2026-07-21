---
title: "Hermeticity: Reproducible Builds"
category: "concepts"
level: "advanced"
status: "growing"
sources: ["Hermeticity    Bazel.md"]
tags: ["hermeticity", "reproducibility", "isolation", "caching"]
graph-group: "concepts-advanced"
related: ["[[concepts/fundamentals/build.bazel]]", "[[troubleshooting/build-failures]]", "[[patterns/dependency-management]]"]
last_updated: "2026-07-19"
---

# Hermeticity: Reproducible Builds

**Hermeticity** is the property that a build system always produces the same output when given the same input source code and configuration.

## Definition

A hermetic build system is **self-contained and isolated** from the host machine environment. When you run the same build twice with identical inputs, you get bit-for-bit identical outputs—regardless of what tools or libraries are installed on your local or remote machine.

## Two Pillars of Hermeticity

### 1. **Isolation**

Hermetic builds treat tools and dependencies as source code, not host machine resources:

- **Download specific tool versions** — Don't rely on `/usr/bin/gcc`; download the exact compiler version needed
- **Manage tools in build trees** — Tools stay in controlled directories, isolated from the host system
- **No implicit system dependencies** — If you need something, declare it explicitly

**Result:** Your build doesn't break when the host system updates, installs new libraries, or changes tool versions.

### 2. **Source Identity**

Hermeticity depends on tracking input changes:

- **Hash-based identities** — Git commits use SHA hashes to identify code mutations uniquely
- **Bazel extends this** — Hashes track not just source code but also configuration, tool versions, and build parameters
- **Input + Config = Output** — The same source code + same build configuration always produces the same output

## Major Benefits

### Speed
- Action outputs are cached
- If inputs haven't changed, actions don't re-run
- No wasted computation

### Parallel Execution
- Bazel constructs a dependency DAG of all actions
- Can execute safe-to-run-in-parallel actions concurrently
- Significant speedup on multi-core machines

### Multiple Builds on One Machine
- Different projects can use different tool versions simultaneously
- No conflicts from shared `/usr/local/bin` tools
- Each build is self-contained

### Reproducibility & Debugging
- You know *exactly* what conditions produced a build
- Easier to troubleshoot and reproduce issues
- Others can reproduce your build results

## Common Sources of Non-Hermeticity

**Non-hermetic builds** leak information from the host machine into the build, causing different outputs from the same inputs:

- **Timestamps & Build IDs** — Actions that include `$(date)` or sequential IDs are non-deterministic
- **System Binaries** — Relying on `/usr/bin` tools that differ across machines
- **Absolute Paths** — Hardcoding `/usr/local/gcc` instead of downloaded, versioned compiler
- **Compiler Autodetection** — Native C++ rules detecting the "current" system compiler (varies by machine)
- **Writing to Source Tree** — Building target A modifies source files, breaking builds of target B on the same tree
- **Arbitrary Scripts** — `.mk` files or custom scripts with implicit system dependencies

## Detecting Non-Hermeticity Issues

### Null Sequential Builds
Run a build, then run the same build again without changing inputs:

```bash
$ bazel build //...    # First build
$ bazel build //...    # Second build (should rebuild nothing)
```

If the second build rebuilds anything, it's non-hermetic.

### Cross-System Validation
Build on multiple machines and compare outputs:

```bash
# On machine A
$ bazel build //my:app
$ sha256sum bazel-bin/my/app

# On machine B (same code, same config)
$ bazel build //my:app
$ sha256sum bazel-bin/my/app
```

If SHA values differ, the build is non-hermetic.

### Docker Sandbox Testing
Run builds in a minimal Docker container:

```dockerfile
FROM alpine:latest
COPY . /workspace
WORKDIR /workspace
RUN bazel build //...
```

If the build fails in a bare container, you have implicit system dependencies.

## Fixing Non-Hermetic Builds

### Enable Sandboxing
Bazel can sandbox actions to catch implicit system dependencies:

```bash
bazel build --sandbox_debug ...
```

Sandboxing isolates each action in a minimal filesystem, revealing dependencies on system resources.

### Use Remote Execution Rules
Bazel provides `@bazel_tools//tools/cpp:test_wrapper` and other helpers to test hermeticity.

### Log Workspace Rules
To find implicit dependencies in workspace rules:

```bash
bazel build --experimental_workspace_rules_log_file=/tmp/log.txt ...
```

This reveals which workspace rules might be non-hermetic.

### Download All Tools Explicitly
Instead of:
```starlark
# BAD: Relies on host /usr/bin/gcc
cc_toolchain(name = "gcc", compiler = "/usr/bin/gcc")
```

Use:
```starlark
# GOOD: Downloads specific compiler version
http_archive(
    name = "gcc_download",
    urls = ["https://example.com/gcc-11.2.tar.gz"],
)
cc_toolchain(name = "gcc", compiler = ":gcc_download")
```

### Avoid Timestamps and Non-Determinism
Don't include build timestamps or sequence numbers in artifacts:

```starlark
# BAD
genrule(
    name = "version",
    cmd = "date > $@",  # Non-deterministic!
)

# GOOD
genrule(
    name = "version",
    cmd = "echo '1.0.0' > $@",  # Deterministic
)
```

## Hermeticity in Bazel

Bazel is *designed* for hermeticity:

- **Sandbox actions** — Each action runs in an isolated filesystem
- **Hash-based caching** — Actions are cached by input hash, not timestamp
- **Version-locked dependencies** — `MODULE.bazel` pins tool and dependency versions
- **Starlark restrictions** — BUILD files can't call arbitrary shell commands (forces explicit dependencies)
- **Remote execution** — Different machines can safely execute the same build

## Best Practices

1. **Declare all dependencies** — If an action needs a tool or library, declare it explicitly
2. **Pin tool versions** — Use versioned downloads, not system tools
3. **Test hermetic builds locally** — Run builds twice; they should produce identical output
4. **Use sandboxing** — Enable `--sandbox_debug` during development
5. **Avoid system assumptions** — Don't assume `/usr/bin/python` or similar
6. **Document custom toolchains** — If you define toolchains, explain their hermeticity properties

## Related Resources

- [Bazel Sandboxing](https://bazel.build/docs/sandboxing)
- [Remote Execution](https://bazel.build/remote/rbe)
- [Remote Caching](https://bazel.build/remote/cache)
- [BazelCon: Building Real-time Systems with Bazel (SpaceX)](https://www.youtube.com/watch?v=t_3bckhV_YI)

---

**Key Takeaway:** Hermetic builds are the foundation of Bazel's speed and reliability. Invest in eliminating non-hermeticity early in your project.
