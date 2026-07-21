---
title: "Calling Bazel from Scripts"
category: "reference"
level: "intermediate"
status: "seedling"
sources: ["Calling Bazel from scripts.md"]
tags: ["#scripting", "#automation", "#cli", "#exit-codes"]
related: ["[[reference/cli-reference]]", "[[reference/bazelrc-configuration]]", "[[reference/remote-execution]]"]
last_updated: "2026-07-19"
graph-group: "reference"
---

# Calling Bazel from Scripts

Bazel is designed to be scriptable, allowing automation of builds, tests, and queries from shell scripts and other tools.

## Basic Script Patterns

### Simple Build

```bash
#!/bin/bash
bazel build //myapp:app
if [ $? -ne 0 ]; then
    echo "Build failed"
    exit 1
fi
```

### Multiple Targets

```bash
#!/bin/bash
bazel build //lib:lib1 //lib:lib2 //app:app || exit 1
```

### Run with Arguments

```bash
#!/bin/bash
bazel run //myapp:app -- --flag1 value1 --flag2 value2
```

### Query for Targets

```bash
#!/bin/bash
# Find all test targets
bazel query 'kind(.*test, //...)'

# Find all libraries that a target depends on
bazel query 'deps(//app:main)'
```

## Managing the Bazel Server

Bazel uses a long-running server process for performance. When scripting:

### Shutdown Server

Clean up when done:

```bash
#!/bin/bash
bazel build //app:app
RESULT=$?
bazel shutdown  # Explicitly shut down server
exit $RESULT
```

### Auto-Shutdown Idle

Automatically shutdown idle servers:

```bash
bazel build //app:app --max_idle_secs=5
```

This prevents servers from lingering indefinitely.

### Multiple Server Instances

For concurrent script execution, use separate `--output_base` directories:

```bash
#!/bin/bash
SCRIPT_INSTANCE=$1
OUTPUT_BASE="/tmp/bazel_build_$SCRIPT_INSTANCE"

bazel build //app:app --output_base="$OUTPUT_BASE"
bazel shutdown --output_base="$OUTPUT_BASE"
```

Each instance gets its own server and lock.

## Output Base Considerations

### Default Behavior

By default, all Bazel commands use the same output base (typically `~/.cache/bazel`). This means:
- All `bazel` commands contend for the same lock
- Script execution waits for user's interactive commands to complete
- Multiple script instances block each other

### Custom Output Base

```bash
#!/bin/bash
bazel build //app:app --output_base=/tmp/my_build
```

Use when:
- You need isolated builds (CI/CD)
- You want to run multiple builds concurrently
- Build artifacts must go to a specific location

### No-Block Mode

If another Bazel process holds the lock:

```bash
#!/bin/bash
bazel build //app:app --noblock_for_lock
```

Exit code 9 indicates the lock is held and `--noblock_for_lock` was used.

## Exit Codes

Understanding exit codes helps scripts react appropriately to failures.

### Common Exit Codes (All Commands)

| Code | Meaning | Action |
|------|---------|--------|
| 0 | Success | Continue |
| 1 | General failure | Retry or fail |
| 2 | Bad command line / flags | Fix command line |
| 3 | Partial success (query) or tests failed | Check results |
| 7 | Query command failure | Fix query |
| 8 | Build interrupted | Orderly shutdown |
| 9 | Server lock held (with `--noblock_for_lock`) | Retry later |
| 32 | External environment failure | Check environment |
| 33 | Out of memory | Increase heap |
| 36 | Local environmental issue | Fix environment |
| 37 | Internal Bazel error | Report bug |

### Build/Test-Specific Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Build failed |
| 3 | Build OK, but tests failed or timed out |
| 4 | Build successful, but no tests found |

### Run-Specific Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | Build failed |
| N | Exit code of the executed subprocess |

### Query-Specific Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 3 | Partial success (errors in BUILD files, with `--keep_going`) |
| 7 | Command failure |

### Example: Script with Exit Code Handling

```bash
#!/bin/bash
set -e  # Exit on any error

bazel build //app:app
BUILD_CODE=$?

case $BUILD_CODE in
    0)
        echo "Build succeeded"
        ;;
    1)
        echo "Build failed"
        exit 1
        ;;
    3)
        echo "Build OK but tests failed"
        exit 1
        ;;
    9)
        echo "Bazel locked, retrying..."
        sleep 5
        exec "$0"  # Retry
        ;;
    *)
        echo "Unexpected exit code: $BUILD_CODE"
        exit 1
        ;;
esac
```

## .bazelrc Configuration

### Default Behavior

By default, Bazel reads `.bazelrc` from:
1. Workspace root directory
2. User's home directory (`~/.bazelrc`)

This means user preferences are applied to scripts automatically.

### Disable .bazelrc for Hermetic Builds

```bash
#!/bin/bash
# CI/release build: ignore user's .bazelrc
bazel build //app:app --bazelrc=/dev/null
```

This ensures reproducible, hermetic builds regardless of user configuration.

### Enable Specific Config

```bash
#!/bin/bash
# Use only a specific config
bazel build //app:app --bazelrc=/etc/ci.bazelrc
```

## Accessing Build Outputs

### Find Output Location

```bash
#!/bin/bash
OUTPUT=$(bazel info bazel-bin)
echo "Build outputs in: $OUTPUT"
```

### Use Generated Artifacts

```bash
#!/bin/bash
bazel build //app:app
BINARY="$(bazel info bazel-bin)/app/app"
"$BINARY" --arg1 value1
```

## Accessing Logs

### Command Log

Get the most recent Bazel command output:

```bash
#!/bin/bash
bazel info command_log
```

Returns the path to the command log file (contains interleaved stdout/stderr).

### Example: Log Inspection

```bash
#!/bin/bash
bazel build //app:app || {
    echo "Build failed. See command log:"
    cat "$(bazel info command_log)"
    exit 1
}
```

## Debugging Scripts

### Verbose Output

```bash
bazel build //app:app -v
```

### Action Output

See what commands were executed:

```bash
bazel build //app:app --verbose_failures
```

### Debugging Specific Target

```bash
bazel build //app:app --explain=explain.log
```

Generates detailed explanation of build decisions.

## Best Practices

### 1. Always Handle Exit Codes

```bash
#!/bin/bash
bazel build //app:app || exit 1
```

### 2. Manage Server Lifecycle

```bash
#!/bin/bash
function cleanup() {
    bazel shutdown
}
trap cleanup EXIT

bazel build //app:app
```

### 3. Use Isolated Output Base for Concurrent Builds

```bash
#!/bin/bash
OUTPUT_BASE="/tmp/bazel_build_$$"  # Use PID for uniqueness
bazel build //app:app --output_base="$OUTPUT_BASE"
```

### 4. Document Config

```bash
#!/bin/bash
# Use CI-specific .bazelrc for reproducible builds
bazel build //app:app --bazelrc=/etc/ci.bazelrc
```

### 5. Add Timeouts

```bash
#!/bin/bash
timeout 300 bazel build //app:app || exit 1
```

## Common Script Patterns

### CI/CD Build

```bash
#!/bin/bash
set -e

# Ensure clean build
bazel clean

# Build release binary
bazel build //app:app \
    -c opt \
    --bazelrc=/dev/null \
    --output_base=/tmp/release_build

# Run tests
bazel test //... \
    -c opt \
    --bazelrc=/dev/null \
    --output_base=/tmp/release_build

# Cleanup
bazel shutdown --output_base=/tmp/release_build
```

### Development Build with User Config

```bash
#!/bin/bash
# Use user's .bazelrc (includes local preferences)
bazel build //app:app
bazel test //app:... --test_output=all
```

### Parallel Builds with Multiple Instances

```bash
#!/bin/bash
NUM_JOBS=4
for i in $(seq 1 $NUM_JOBS); do
    (
        OUTPUT_BASE="/tmp/bazel_job_$i"
        bazel build //app_$i:app --output_base="$OUTPUT_BASE"
    ) &
done
wait
```

## See Also

- [[reference/cli-reference]] — Complete Bazel CLI documentation
- [[reference/bazelrc-configuration]] — .bazelrc configuration options
- [[reference/remote-execution]] — Remote execution and scripting
