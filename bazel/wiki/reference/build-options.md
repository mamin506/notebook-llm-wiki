---
title: "Build Options Reference"
category: "reference"
level: "intermediate"
status: "growing"
sources: ["Commands and Options.md"]
tags: ["cli", "build", "configuration"]
related: ["[[reference/cli-reference]]", "[[concepts/fundamentals/build.bazel]]", "[[concepts/advanced/platforms]]"]
last_updated: "2026-07-19"
graph-group: "reference"
---

# Build Options Reference

Deep dive into Bazel's build-time options and their effects.

## Compilation Modes

The `--compilation_mode (-c)` flag controls optimization level and debug info:

| Mode | Flags | Use Case |
|------|-------|----------|
| **fastbuild** (default) | `-gmlt -Wl,-S` no optimization | Fastest compilation, dev builds |
| **dbg** | `-g` with debug symbols | Debugging, inspecting state |
| **opt** | `-O2 -DNDEBUG` | Release builds, performance-critical |

Example:
```bash
bazel build -c opt //foo:binary  # Optimized release build
bazel build -c dbg //foo:binary  # Debug-enabled build
```

## Compiler & Linker Options

### C/C++ Compilation

- `--copt=flag` — Option passed to all C/C++ compilations
- `--conlyopt=flag` — Option for C compilation only
- `--cxxopt=flag` — Option for C++ compilation only
- `--linkopt=flag` — Option passed to linker
- `--per_file_copt=pattern@options` — Per-file compiler options

Example:
```bash
bazel build --cxxopt="-std=c++17" --linkopt="-lm" //foo
```

### Position-Independent Code

- `--[no]force_pic` — Force position-independent code (default: off)

Useful for shared libraries and PIE executables:
```bash
bazel build --force_pic //foo:shared_lib
```

### Dynamic Linking

- `--dynamic_mode=mode` — How C++ binaries are linked
  - `default` — Bazel decides
  - `fully` — All dynamic (faster links, smaller binaries)
  - `off` — Mostly static

```bash
bazel build --dynamic_mode=fully //foo:binary
```

## Execution Strategies

The `--spawn_strategy` flag controls how actions are executed:

| Strategy | Pros | Cons |
|----------|------|------|
| `sandboxed` | Maximum hermeticity, reproducible | Slower (creates sandbox) |
| `local` | Faster for interactive dev | Less hermetic |
| `worker` | Fast for repeated actions | Stateful (can cause issues) |
| `docker` | Consistent cross-machine | Requires Docker |
| `remote` | Massive parallelism | Requires RBE setup |

Example:
```bash
bazel build --spawn_strategy=docker //foo  # Docker sandbox
```

Per-action strategy override:
```bash
bazel build --strategy=Genrule=local //foo  # Use local for genrules
```

## Job Control

- `--jobs=N (-j)` — Number of parallel actions
  - Default: auto-detected from CPU count
  - Typical: `--jobs=8` on 8-core machine
  - For CI: often `--jobs=4` to avoid overwhelming server

```bash
bazel build --jobs=4 //foo  # Limit to 4 parallel jobs
```

## Environment Variables

### For Build Actions

- `--action_env=VAR=value` — Make variable available to all actions
- `--action_env=VAR` — Inherit from invocation environment

Example:
```bash
bazel build --action_env=CUSTOM_FLAG=1 //foo
```

### For Repository Rules

- `--repo_env=VAR=value` — Environment for repository rule execution

Example:
```bash
bazel build --repo_env=HTTP_PROXY=http://proxy:8080 //foo
```

## Stamping & Build Info

### Workspace Status

The `--workspace_status_command=program` flag lets you inject build metadata:

1. Create a script that outputs `KEY value` pairs:
```bash
#!/bin/bash
echo "STABLE_COMMIT $(git rev-parse HEAD)"
echo "BUILD_TIMESTAMP $(date +%s)"
```

2. Pass to Bazel:
```bash
bazel build --workspace_status_command=/path/to/script.sh //foo
```

3. Access in rules via `$(BUILD_COMMIT)`, `$(BUILD_TIMESTAMP)`, etc.

### Stamping Control

- `--[no]stamp` — Enable/disable stamping (default: auto, depends on rule)
  - `stamp=1` — Always stamp
  - `stamp=0` — Never stamp
  - `stamp=-1` — Use `--stamp` flag to decide

```bash
bazel build --stamp //foo:release_binary
```

## Testing Options

### Test Caching

- `--cache_test_results=(yes|no|auto)` — Cache test results
  - `auto` (default) — Cache if test unchanged
  - `yes` — Cache even if sources changed (use carefully!)
  - `no` — Always run

```bash
bazel test --cache_test_results=no //foo:tests  # Always run
```

### Test Filtering

- `--test_size_filters=small,medium` — Only run small/medium tests
- `--test_timeout_filters=short,moderate` — Only run quick tests
- `--test_tag_filters=integration,-flaky` — Include/exclude by tag
- `--test_filter=regex` — Filter test cases by name

Example:
```bash
bazel test --test_size_filters=small //foo:all_tests
```

### Test Environment

- `--test_env=VAR=value` — Inject environment variable for tests
- `--test_tmpdir=path` — Temporary directory for test execution
- `--test_arg=arg` — Pass argument to each test

```bash
bazel test --test_env=DEBUG=1 --test_tmpdir=/tmp/test //foo:tests
```

## Output Control

### Output Filtering

- `--output_filter=regex` — Only show output for matching targets

Example:
```bash
bazel build --output_filter='^//foo:' //foo/... //bar/...
```

This shows output only for `//foo/...`, suppressing `//bar/...` logs.

### Result Display

- `--show_result=n` — Show result info for first `n` targets (default: 1)
  - `0` — Never show
  - Large number — Always show

```bash
bazel build --show_result=5 //foo:...  # Show first 5 results
```

## Advanced Options

### Genrule & Extra Actions (Advanced)

- `--experimental_action_listener=label` — Insert extra actions
- `--experimental_extra_action_filter=regex` — Filter extra actions

### Android-Specific

- `--android_platforms=platform[,platform]*` — Platforms for Android native deps
- `--android_resource_shrinking` — Enable resource shrinking

### Java-Specific

- `--java_language_version=version` — Java language version (8, 11, 17, 21)
- `--java_runtime_version=version` — JVM version for execution
- `--strict_java_deps` — Check for missing direct dependencies

Example:
```bash
bazel build --java_language_version=11 //foo:java_app
```

---

## Tips

1. **Reproducible builds:** Use `--compilation_mode=opt --stamp` for releases
2. **Debug builds:** Use `--compilation_mode=dbg` to keep debug symbols
3. **Faster local dev:** Use `--compilation_mode=fastbuild` (default)
4. **Hermetic builds:** Use `--spawn_strategy=sandboxed` (but slower)
5. **CI/Release:** Use `--bazelrc=/dev/null` to avoid local overrides
