---
title: "Bazel CLI Reference"
category: "reference"
level: "intermediate"
status: "growing"
sources: ["Commands and Options.md", "Command-Line Reference.md"]
tags: ["cli", "commands", "#recommended"]
related: ["[[concepts/fundamentals/targets]]", "[[reference/build-options]]", "[[reference/build-command-options]]", "[[patterns/monorepo-layout]]"]
last_updated: "2026-07-29"
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
bazel build -c opt --jobs=8 //app:binary
```

**Key options:**
- `--compilation_mode (-c)` — fastbuild (default), dbg, opt
- `--jobs (-j)` — Number of parallel jobs (default: auto-detected)
- `--keep_going (-k)` — Continue building even after errors
- `--output_filter=regex` — Filter output by regex

**See also:** [[reference/build-command-options]] for comprehensive build option reference

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

### bazel aquery
Analyzes the given targets and queries the action graph (action-level details).

```bash
bazel aquery 'action(//foo:app)'
bazel aquery --output=text 'filter("JavaCompile", //foo:app)'
```

### bazel cquery
Loads, analyzes, and queries the specified targets with configurations (config-aware).

```bash
bazel cquery --output=graph 'deps(//foo)' --cpu=aarch64
bazel cquery 'attr("tags", "manual", //...)'
```

### bazel query
Executes a dependency graph query (operates after loading phase).

```bash
bazel query 'deps(//foo)'
bazel query 'rdeps(//app, //lib)'
```

**Differences:**
- `query` — Fast, operates on loaded targets only
- `cquery` — Slower, respects build flags (--cpu, --platforms, etc.)
- `aquery` — Operates on action graph, shows actual compilation commands

### bazel fetch
Fetches external repositories (prerequisites to targets).

```bash
bazel fetch //app:all
bazel fetch --repo=@python_interpreter
```

### bazel coverage
Generates code coverage report for specified test targets.

```bash
bazel coverage //app:app_test --coverage_report_generator=@bazel_tools//tools/cpp:coverage_generator
```

### bazel mobile-install
Installs targets to mobile devices (iOS/Android).

```bash
bazel mobile-install //app:ios_app
bazel mobile-install --device=device_id //app:android_app
```

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

## Utility & Diagnostic Commands

### bazel info
Display Bazel configuration and runtime info.

```bash
bazel info workspace         # Workspace root directory
bazel info output_base       # Output base directory
bazel info bazel-bin         # Build output directory
bazel info server_pid        # Bazel server process ID
bazel info all               # Show all info
```

### bazel version
Display Bazel version information.

```bash
bazel version
bazel --version              # Shorter form, doesn't start server
```

### bazel help
Show help for commands and topics.

```bash
bazel help build
bazel help --long build      # Detailed help with defaults
bazel help                   # Show all available commands
```

### bazel dump
Dumps the internal state of the Bazel server process.

```bash
bazel dump --skyframe_state=/tmp/skyframe.txt
```

### bazel shutdown
Stop the Bazel server.

```bash
bazel shutdown
bazel shutdown --iff_heap_size_greater_than 512  # Conditional shutdown
```

### bazel print_action
Prints the command-line args for compiling a file.

```bash
bazel print_action //app:main.cc
```

### bazel mod
Queries the Bzlmod external dependency graph (module system).

```bash
bazel mod graph --extension=@bazel_tools//extensions:python
bazel mod query
```

### bazel vendor
Fetches external repositories into a folder.

```bash
bazel vendor --vendor_dir=vendor
```

### bazel canonicalize-flags
Canonicalizes a list of Bazel options (useful for build scripts).

```bash
bazel canonicalize-flags --source_root=/workspace --build_root=/tmp
```

### bazel license
Prints the license of this software.

```bash
bazel license
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
