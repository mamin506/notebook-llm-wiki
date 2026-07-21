---
title: "Bazel CLI Reference"
category: "reference"
level: "intermediate"
status: "growing"
sources: ["Commands and Options.md"]
tags: ["cli", "commands", "#recommended"]
related: ["[[concepts/fundamentals/targets]]", "[[reference/build-options]]", "[[patterns/monorepo-layout]]"]
last_updated: "2026-07-19"
graph-group: "reference"
---

# Bazel CLI Reference

Complete reference for Bazel commands and command-line options.

## Core Commands

### bazel build
Builds one or more targets.

```bash
bazel build //foo
bazel build //foo/... --compilation_mode=opt
```

**Key options:**
- `--compilation_mode (-c)` — fastbuild (default), dbg, opt
- `--jobs (-j)` — Number of parallel jobs (default: auto-detected)
- `--keep_going (-k)` — Continue building even after errors
- `--output_filter=regex` — Filter output by regex

### bazel test
Builds and runs tests.

```bash
bazel test //foo:all_tests
bazel test --test_filter="TestClass#testMethod" //foo/...
```

**Key options:**
- `--cache_test_results` — auto (default), yes, no
- `--test_tag_filters` — Filter by test tags
- `--test_size_filters` — Filter by test size (small, medium, large, enormous)
- `--test_timeout_filters` — Filter by timeout (short, moderate, long, eternal)
- `--runs_per_test=N` — Run each test N times
- `--test_output` — summary (default), errors, all, streamed

### bazel run
Builds and runs a single executable target.

```bash
bazel run //foo:app -- --arg1 --arg2
```

**Note:** Arguments after `--` are passed to the target, not Bazel.

### bazel query / bazel cquery
Query the dependency graph.

```bash
bazel query 'deps(//foo)'
bazel cquery --output=graph 'deps(//foo)' --cpu=aarch64
```

- `query` — operates after loading phase
- `cquery` — operates after analysis phase (respects build flags)

### bazel clean
Removes build outputs.

```bash
bazel clean              # Clean output directories
bazel clean --expunge    # Remove entire output base
bazel clean --expunge_async  # Async expunge
```

## Common Options

### Compilation & Linking

- `--copt=flag` — Pass option to C++ compiler
- `--conlyopt=flag` — Pass option to C compiler (not C++)
- `--cxxopt=flag` — Pass option to C++ compiler
- `--linkopt=flag` — Pass option to linker
- `--strip` — Strip binaries (always|never|sometimes)

### Platform & Toolchain

- `--platforms=label` — Target platform (modern, recommended)
- `--cpu=cpu` — Target CPU architecture (legacy, use --platforms)
- `--host_platform=label` — Host platform
- `--extra_toolchains=labels` — Additional toolchains to consider

### Execution Strategy

- `--spawn_strategy=strategy` — How to execute actions
  - `sandboxed` — Local sandbox (default on most systems)
  - `local` — Direct subprocess execution
  - `worker` — Persistent worker process
  - `docker` — Docker sandbox
  - `remote` — Remote execution

### Output & Artifacts

- `--symlink_prefix=string` — Prefix for symlinks (default: `bazel-`)
- `--output_user_root=dir` — Root for output and install bases
- `--show_result=n` — Show result info for first n targets

### Logging & Debugging

- `--explain=logfile` — Explain why actions were executed
- `--profile=file` — Write build profile (JSON trace format)
- `--subcommands (-s)` — Print full command lines before executing
- `--verbose_failures` — Print full command lines for failed actions
- `--sandbox_debug` — Print sandbox debugging info

### Test Options

- `--java_debug` — Wait for debugger on Java tests
- `--test_tmpdir=path` — Temporary directory for test execution
- `--test_env=VAR=value` — Inject environment variables
- `--test_arg=arg` — Pass arguments to each test

### Configuration & Environment

- `--action_env=VAR=value` — Make variable available in action environment
- `--config=name` — Select config section from `.bazelrc`
- `--bazelrc=path` — Use specific `.bazelrc` file (default: search standard locations)
- `--repo_env=VAR=value` — Environment variable for repository rules

## Startup Options

Startup options affect the Bazel server and must appear before the command:

```bash
bazel --output_base=/tmp/my_bazel_output build //foo
```

- `--output_base=dir` — Where to write all output
- `--output_user_root=dir` — Root for output/install bases
- `--server_javabase=dir` — JDK/JRE for Bazel itself
- `--host_jvm_args=string` — JVM options for Bazel server
- `--batch` — Run single command instead of client/server
- `--max_idle_secs=n` — Server idle timeout (default: 10800 / 3 hours)

## Miscellaneous

### info
Display Bazel configuration info.

```bash
bazel info workspace         # Workspace root directory
bazel info output_base       # Output base directory
bazel info bazel-bin         # Build output directory
bazel info server_pid        # Bazel server process ID
```

### version
Display Bazel version information.

```bash
bazel version
bazel --version  # Shorter form, doesn't start server
```

### help
Show help for commands and topics.

```bash
bazel help build
bazel help --long build  # Detailed help with defaults
```

### shutdown
Stop the Bazel server.

```bash
bazel shutdown
bazel shutdown --iff_heap_size_greater_than 512  # Conditional shutdown
```

---

## Performance Tips

- Use `--jobs=N` to control parallelism (default is usually optimal)
- Use `--keep_going (-k)` during development to find all errors at once
- Use `--explain=logfile` to debug unexpected rebuilds
- Profile slow builds with `--profile=file` + [JSON trace profile](https://bazel.build/advanced/performance/json-trace-profile)
- For releases, use `--bazelrc=/dev/null` to ignore local config

---

## Release Build Recommendations

When using Bazel in scripts (especially CI/release builds):

```bash
bazel build --bazelrc=/dev/null --nokeep_state_after_build //foo
```

- `--bazelrc=/dev/null` — Ignore local config files
- `--nokeep_state_after_build` — Clean up state between builds
- `--symlink_prefix=/` — Suppress symlink creation if desired
