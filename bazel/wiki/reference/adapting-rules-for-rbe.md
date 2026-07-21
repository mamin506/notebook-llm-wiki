---
title: "Adapting Custom Rules for Remote Execution"
category: "reference"
level: "advanced"
status: "seedling"
sources: ["Adapting Bazel Rules for Remote Execution.md"]
tags: ["#remote-execution", "#rbe", "#custom-rules", "#toolchains"]
related: ["[[reference/remote-execution]]", "[[concepts/advanced/hermeticity]]", "[[reference/platforms-toolchains-rules.md]]"]
last_updated: "2026-07-19"
graph-group: "reference"
---

# Adapting Custom Rules for Remote Execution

When writing custom Bazel rules that will run with remote execution (RBE), special considerations apply to ensure rules work correctly in a remote, isolated environment.

## Overview

Remote execution differs from local execution in key ways:
1. **Isolation** — Each action runs in its own sandbox with only declared inputs
2. **No Shared State** — Tools can't retain state across actions
3. **Diverse Environments** — Remote platforms may differ from local machine
4. **No Implicit Dependencies** — Every tool and dependency must be explicitly declared

These constraints require careful rule design to avoid failures in remote execution.

## Core Requirements

### 1. Use Toolchain Rules

**Problem:** Invoking tools via `PATH`, environment variables, or absolute paths fails remotely because those tools/paths may not exist on remote machines.

**Solution:** Use Bazel toolchain rules to locate and invoke tools:

```python
# ❌ BAD: Assumes javac in PATH
def java_compile_impl(ctx):
    return java_compile(
        ctx,
        cmd = "javac $@",  # Will fail remotely
    )

# ✅ GOOD: Use toolchain
def java_compile_impl(ctx):
    java_toolchain = ctx.toolchains["@bazel_tools//tools/java:toolchain_type"]
    cmd = "{} $@".format(java_toolchain.javac_path)
    return ctx.actions.run_shell(...)
```

**Toolchains to Use:**
- C++: `@bazel_tools//tools/cpp:toolchain_type`
- Java: `@bazel_tools//tools/java:toolchain_type`
- Python: `@bazel_tools//tools/python:toolchain_type`
- Go: `@bazel_tools//tools/go:toolchain_type`

### 2. Declare All Dependencies

**Problem:** If a tool implicitly depends on files not in its declared inputs, the action fails remotely.

**Solution:** Explicitly declare all inputs and dependencies:

```python
# ❌ BAD: Compiler implicitly finds libc headers
ctx.actions.run(
    executable = compiler,
    inputs = [source_file],  # Missing system libraries
    outputs = [object_file],
    arguments = [source_file, "-o", object_file],
)

# ✅ GOOD: Declare all inputs
ctx.actions.run(
    executable = compiler,
    inputs = depset(
        direct = [source_file],
        transitive = [
            dep[CppInfo].headers for dep in ctx.attr.deps
        ]
    ),
    outputs = [object_file],
    arguments = [source_file, "-o", object_file],
)
```

**Common Implicit Dependencies:**
- System headers (`/usr/include`)
- System libraries (`libc`, `libm`)
- Build tools (compilers, linkers)
- Configuration files

### 3. Avoid Platform-Specific Binaries

**Problem:** A binary built for your local machine (e.g., Linux x86_64) won't run on a remote machine with a different architecture.

**Solution:** Either:
- Ship source code and build the tool as part of the build
- Use platform-independent tools (scripts, Java, Python)
- Declare the tool via `tools` and let Bazel build it for the remote platform

```python
# ❌ BAD: Ship pre-built binary
ctx.actions.run(
    executable = "bin/my_tool_linux",  # Only runs on Linux
    inputs = [source_file],
    outputs = [output_file],
)

# ✅ GOOD: Reference a buildable target
ctx.actions.run(
    executable = ctx.attr.tool,  # Will be built for remote platform
    inputs = depset(direct = [source_file]),
    outputs = [output_file],
    tools = [ctx.attr.tool],
)
```

Declare in the rule:
```python
my_rule = rule(
    implementation = my_rule_impl,
    attrs = {
        "tool": attr.label(
            executable = True,
            cfg = "exec",  # Build for execution platform
        ),
    },
)
```

### 4. Avoid Stateful Tools

**Problem:** Some tools retain state across invocations. When actions run in separate containers remotely, this state is lost.

**Solution:** Only use tools that process each action independently:

```python
# ❌ BAD: Compiler with implicit caching
ctx.actions.run(
    executable = stateful_compiler,
    inputs = [foo_cc],
    outputs = [foo_o],
)
# Second action uses foo_o but compiler doesn't know about foo_cc
ctx.actions.run(
    executable = stateful_compiler,
    inputs = [bar_cc],  # Compiler lost state from foo_cc
    outputs = [bar_o],
)

# ✅ GOOD: Stateless compiler
ctx.actions.run(
    executable = stateless_compiler,
    inputs = depset(direct = [foo_cc]),
    outputs = [foo_o],
)
ctx.actions.run(
    executable = stateless_compiler,
    inputs = depset(direct = [bar_cc]),
    outputs = [bar_o],
)
```

## Common Adaptations

### Docker Sandbox Testing

Before using a rule with RBE, test locally with Docker sandbox to catch isolation issues:

```bash
bazel build //mylib:lib \
    --spawn_strategy=docker \
    --extra_docker_opts=--net=none  # Disable network access
```

This simulates remote execution's isolation locally.

### Environment Variables

Avoid relying on environment variables; explicitly declare what's needed:

```python
# ❌ BAD: Relies on JAVA_HOME
cmd = "$JAVA_HOME/bin/java $@"

# ✅ GOOD: Use toolchain or explicit path
java_toolchain = ctx.toolchains["@bazel_tools//tools/java:toolchain_type"]
cmd = "{} $@".format(java_toolchain.java_path)
```

### Build Flags and Options

Don't hardcode build flags; pass them through attributes:

```python
# ❌ BAD: Hardcoded flags
cmd = "compiler -O2 -Wall"

# ✅ GOOD: Configurable
cmd = "compiler {} {}".format(
    " ".join(ctx.attr.copts),
    input_file,
)
```

### Temporary Files

Avoid writing to fixed paths; use Bazel-managed directories:

```python
# ❌ BAD: Fixed temp directory
ctx.actions.run(
    cmd = "cmd > /tmp/output.txt",
)

# ✅ GOOD: Use Bazel output file
temp_file = ctx.actions.declare_file("temp.txt")
ctx.actions.run(
    outputs = [temp_file],
    cmd = "cmd > " + temp_file.path,
)
```

## Advanced Patterns

### Tool Wrapper Scripts

Create wrapper scripts that handle platform-specific logic:

```python
# wrapper.sh
#!/bin/bash
TOOLCHAIN_PATH="$(dirname "$0")/toolchain"
export PATH="$TOOLCHAIN_PATH:$PATH"
exec mycompiler "$@"
```

Then in the rule:
```python
ctx.actions.run(
    executable = wrapper_script,
    tools = [toolchain_dir],
)
```

### Remote Platform Properties

Specify platform-specific properties for remote execution:

```python
platform(
    name = "rbe_platform",
    constraint_values = [
        "@platforms//os:linux",
        "@platforms//cpu:x86_64",
    ],
    exec_properties = {
        "docker-image": "gcr.io/company/bazel:ubuntu20.04",
        "container-cpus": "4",
        "container-mem": "8g",
    },
)

# Build with:
# bazel build //app:app --platforms=//platforms:rbe_platform \
#     --remote_executor=grpcs://rbe.company.com
```

### Deterministic Outputs

Ensure outputs are deterministic (same inputs = same outputs):

```python
# ❌ BAD: Includes timestamp
cmd = "echo $(date) > output.txt"

# ✅ GOOD: Deterministic
cmd = "echo 'v1.0.0' > output.txt"

# ✅ GOOD: Includes source hash, not timestamp
cmd = "sha256sum input.txt > output.txt"
```

## Validation Checklist

When adapting rules for RBE:

- [ ] All tools invoked via toolchain rules, not `PATH`
- [ ] All file dependencies explicitly declared in `inputs`
- [ ] No environment variable assumptions
- [ ] No hardcoded absolute paths
- [ ] No temporary files in fixed locations
- [ ] Tools don't retain state across actions
- [ ] Binaries either platform-independent or built from source
- [ ] Build outputs are deterministic
- [ ] Tested with `--spawn_strategy=docker` locally
- [ ] Tested with actual RBE (if available)

## Testing RBE Compatibility

### Local Testing

```bash
# Test with Docker sandbox (simulates RBE isolation)
bazel build //app:app --spawn_strategy=docker

# Test with no network (simulates RBE)
bazel build //app:app --spawn_strategy=docker \
    --extra_docker_opts=--net=none

# Test specific action
bazel build //app:app -s | grep "compile" | head -1
```

### Remote Testing

```bash
# Build with actual RBE
bazel build //app:app \
    --spawn_strategy=remote \
    --remote_executor=grpcs://rbe.example.com

# Check exit code for success (0) or specific errors (9, 32, 36, 37)
echo $?
```

## Troubleshooting

**Issue: "file not found" error in remote execution**
- Ensure file is declared in rule's `inputs` or `srcs`
- Check that dependencies are transitively included
- Verify file paths are relative, not absolute

**Issue: "tool not found"**
- Use toolchain rule instead of PATH
- Declare tool in `tools` attribute
- Ensure tool is built for remote platform

**Issue: Different output on remote vs local**
- Check for timestamps, UUIDs, or non-determinism
- Verify all inputs are included
- Test with `--spawn_strategy=docker` locally

**Issue: "permission denied" executing tool**
- Ensure tool is marked executable
- Check that tool is a valid binary for remote platform
- Verify no file permission issues

## See Also

- [[reference/remote-execution]] — Remote execution overview and configuration
- [[concepts/advanced/hermeticity]] — Hermetic, reproducible builds
- [[reference/platforms-toolchains-rules.md]] — Platform and toolchain configuration
- [[reference/cli-reference]] — `--spawn_strategy` and RBE flags
