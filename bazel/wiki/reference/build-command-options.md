---
title: "Build Command Options Reference"
category: "reference"
level: "intermediate"
status: "growing"
sources: ["Command-Line Reference.md"]
tags: ["build", "cli", "options", "#deep-reference"]
related: ["[[reference/cli-reference]]", "[[reference/build-options]]", "[[reference/execution-tags-and-caching]]"]
last_updated: "2026-07-29"
---

# Build Command Options Reference

Comprehensive reference for all `bazel build` command options. This page complements [[reference/cli-reference]] with detailed coverage.

---

## Compilation & Linking Options

Control how the C/C++ compiler and linker are invoked.

### Compiler Flags

**`--copt=<flag>`**
- Pass an option to the C++ compiler.
- Can be specified multiple times.
- Example: `--copt=-std=c++17`, `--copt=-O3`

**`--conlyopt=<flag>`**
- Pass an option to the C compiler only (not C++).
- Useful for C-specific flags.

**`--cxxopt=<flag>`**
- Pass an option to the C++ compiler (same as `--copt`).

### Linker Flags

**`--linkopt=<flag>`**
- Pass an option to the linker.
- Example: `--linkopt=-Wl,-rpath,/custom/lib`

**`--strip=<always|never|sometimes>`** (default: sometimes)
- Control binary stripping.
- `always` — Always strip binaries.
- `never` — Never strip.
- `sometimes` — Strip based on build mode.

### Build Mode

**`--compilation_mode (-c) <fastbuild|dbg|opt>`** (default: fastbuild)
- Affects optimization level and debug symbols.

| Mode | Optimization | Debug Symbols | Use Case |
|------|--------------|---------------|----------|
| fastbuild | None | Minimal | Development, fast iteration |
| dbg | None | Yes | Debugging |
| opt | Full | Stripped | Production releases |

---

## Platform & Toolchain Options

Select target and host platforms, and configure toolchains.

**`--platforms=<label>`** (recommended, modern)
- Target platform (e.g., `@local_config_platform//:linux_x86_64`).
- Replaces the legacy `--cpu` flag.
- Example: `--platforms=@io_bazel_rules_go//go/toolchain:linux_amd64`

**`--cpu=<cpu_name>`** (legacy, deprecated)
- Target CPU architecture (e.g., `x86_64`, `aarch64`).
- **Prefer `--platforms` in new code.**

**`--host_platform=<label>`**
- Platform for tools that run during the build.
- Usually auto-detected; rarely needs to be set.

**`--extra_toolchains=<labels>`** (multiple)
- Additional toolchains to consider in toolchain resolution.

---

## Execution & Caching Options

Control how Bazel executes actions and uses caches.

### Caching

**`--disk_cache=<path>`**
- Local disk cache directory for action results.
- Use `--disk_cache` (no value) for default location: `<output_user_root>/cache/disk`.

**`--[no]remote_cache=<url>`**
- Remote cache URL (HTTP or gRPC).
- Example: `--remote_cache=https://cache.example.com:8080`
- See [[reference/remote-caching-setup]] for setup details.

**`--[no]remote_upload_local_results`** (default: true)
- Whether to upload locally executed results to remote cache.
- Set to `false` to read-only from remote cache.

**`--[no]remote_verify_downloads`** (default: true)
- Verify hash of remotely downloaded artifacts.

**`--[no]cache_test_results=<yes|no|auto>`** (default: auto)
- Cache test results.
- `auto` — Cache only for tests tagged with `@bazel_tools//tools/test:size_manual`.

### Execution Strategy

**`--spawn_strategy=<strategy>`**
- How to execute actions.

| Strategy | Description | Use Case |
|----------|-------------|----------|
| `sandboxed` | Sandbox isolation (default on Linux) | Hermetic builds |
| `local` | Direct subprocess execution | Debugging, local-only tools |
| `worker` | Persistent worker process | Performance (reduced startup) |
| `docker` | Docker container sandbox | CI/CD environments |
| `remote` | Remote execution | Distributed builds (RBE) |

**`--[no]sandbox`**
- Enable/disable sandboxing (global toggle).

**`--[no]nosandbox`**
- Alias for `--nosandbox`.

### Remote Execution (RBE)

**`--remote_executor=<url>`**
- Remote execution service URL.
- Example: `--remote_executor=grpcs://buildfarm.example.com`

**`--remote_timeout=<duration>`** (default: 60s)
- Maximum time to wait for remote execution/cache calls.
- Format: `<number>[s|m|h|d]` (e.g., `300s`, `5m`)

**`--remote_retries=<integer>`** (default: 5)
- Number of retries for transient remote errors.

**`--remote_retry_max_delay=<duration>`** (default: 5s)
- Maximum backoff delay between retries.

**`--remote_max_connections=<integer>`** (default: 100)
- Maximum concurrent connections to remote cache/executor.

---

## Job Control & Parallelism

**`--jobs (-j) <integer>`** (default: auto-detected)
- Number of parallel build actions.
- Default is usually 2 × CPU count.
- Use `--jobs=1` for sequential builds (debugging).

**`--[no]keep_going (-k)`** (default: false)
- Continue building after errors.
- Useful during development to find all errors at once.

---

## Output & Debugging Options

**`--show_result=<integer>`** (default: 1)
- Show detailed result info for first N targets.

**`--show_timestamps`**
- Prepend timestamp to console output.

**`--[no]symlink_prefix=<prefix>`** (default: `bazel-`)
- Prefix for symlinks to output directories.
- Example: `--symlink_prefix=out-`

**`--explain=<logfile>`**
- Write file explaining why each action was executed.
- Use to debug unexpected rebuilds.

**`--profile=<file>`**
- Write build profile in JSON trace format.
- View with `chrome://tracing` in Chrome/Chromium.

**`--subcommands (-s)`**
- Print full command lines before executing actions.

**`--verbose_failures`**
- Print full command lines for failed actions.

**`--sandbox_debug`**
- Print sandbox debugging information.

**`--output_filter=<regex>`**
- Filter output by regex (hide matching lines).

---

## Test Options

**`--test_tag_filters=<comma-separated tags>`**
- Only run tests with these tags.
- Example: `--test_tag_filters=integration,-slow`

**`--test_size_filters=<sizes>`**
- Filter tests by size: `small`, `medium`, `large`, `enormous`.
- Example: `--test_size_filters=small,medium`

**`--test_timeout_filters=<timeouts>`**
- Filter tests by timeout: `short`, `moderate`, `long`, `eternal`.

**`--runs_per_test=<integer>`**
- Run each test N times (for flakiness detection).

**`--test_output=<output_mode>`**
- Control test output verbosity.

| Mode | Output |
|------|--------|
| `summary` (default) | Only summary |
| `errors` | Only failures |
| `all` | Full output |
| `streamed` | Real-time streaming |

**`--test_env=<VAR=value>`** (multiple)
- Inject environment variables for tests.
- Example: `--test_env=DEBUG=1 --test_env=LOG_LEVEL=INFO`

**`--test_tmpdir=<path>`**
- Temporary directory for test execution.

---

## Configuration & Environment

**`--action_env=<VAR=value>`** (multiple)
- Make environment variable available to all actions.
- Use this instead of directly exporting for hermeticity.

**`--repo_env=<VAR=value>`**
- Environment variable for repository rules.

**`--config=<name>`** (multiple)
- Select config section from `.bazelrc`.
- Example: `--config=ci --config=optimized`

**`--bazelrc=<path>`**
- Use specific `.bazelrc` file (default: search standard locations).

---

## Remote Caching Configuration

### Authentication

**`--google_default_credentials`**
- Use Google Application Default Credentials.
- For Google Cloud Storage cache.

**`--google_credentials=<path>`**
- Path to JSON service account key.

**`--google_auth_scopes=<scopes>`**
- Comma-separated list of OAuth scopes.
- Default: `https://www.googleapis.com/auth/cloud-platform`

### Credential Helpers

**`--credential_helper=<path>`**
- Path to credential helper for authentication (multiple allowed).
- Conforms to [Credential Helper Specification](https://github.com/EngFlow/credential-helper-spec).

**`--credential_helper_timeout=<duration>`** (default: 10s)
- Timeout for credential helper invocation.

---

## Disk & Cache Management

**`--disk_cache=<path>`**
- Local disk cache directory.
- Use `--disk_cache` (no value) for default: `<output_user_root>/cache/disk`.

**`--experimental_disk_cache_gc_max_size=<size>`**
- Maximum disk cache size before garbage collection.
- Format: `<number>[K|M|G|T]` (e.g., `50G`)

**`--experimental_disk_cache_gc_max_age=<duration>`**
- Remove cache entries older than this age.
- Format: `<number>[d|h|m|s]`

**`--experimental_disk_cache_gc_idle_delay=<duration>`** (default: 5m)
- Wait this long after server idle before GC.

---

## Miscellaneous

**`--color=<yes|no|auto>`** (default: auto)
- Colorize output using terminal controls.

**`--curses=<yes|no|auto>`** (default: auto)
- Minimize scrolling with cursor controls.

**`--[no]build_manual_tests`**
- Include tests tagged `manual` in wildcard builds.

**`--[no]keep_state_after_build`**
- Keep Bazel server state after build completes.
- Default is true; useful for interactive use.

**`--nouse_action_cache`**
- Disable action cache (force re-execution).

---

## Common Usage Patterns

### Development Build (Fast, Incremental)
```bash
bazel build -c fastbuild --keep_going //app:app
```

### Release Build (Optimized, Stripped)
```bash
bazel build -c opt --strip=always //app:app
```

### Build with Remote Cache
```bash
bazel build \
  --disk_cache=$HOME/.bazel/cache \
  --remote_cache=https://cache.example.com \
  --google_credentials=$HOME/cache-key.json \
  //app:app
```

### Debug Build (With Symbols)
```bash
bazel build -c dbg --subcommands //app:app
```

### Profile a Slow Build
```bash
bazel build --profile=/tmp/build.json //app:app
# Then open /tmp/build.json in chrome://tracing
```

### Development With Keep Going
```bash
bazel build --keep_going --subcommands -j8 //app/...
```

---

## See Also

- [[reference/cli-reference]] — Overview of all Bazel commands
- [[reference/build-options]] — Older reference (consolidating into this page)
- [[reference/bazelrc-configuration]] — Permanent `.bazelrc` configuration
- [[reference/execution-tags-and-caching]] — Tags for controlling execution behavior
- [[reference/remote-caching-setup]] — Setting up remote caching backends
