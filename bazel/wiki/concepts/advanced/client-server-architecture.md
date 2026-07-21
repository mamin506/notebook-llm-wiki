---
title: "Bazel Client-Server Architecture"
category: "concepts"
level: "advanced"
status: "seedling"
sources: ["Client server implementation.md"]
tags: ["architecture", "internals", "server", "performance", "caching"]
related: ["[[reference/calling-bazel-from-scripts]]", "[[concepts/advanced/hermeticity]]"]
last_updated: "2026-07-20"
---

# Bazel Client-Server Architecture

## Overview

Bazel is implemented as a **long-lived server process**, not a traditional batch-oriented tool. This architecture enables significant optimizations not possible with standalone tools.

```
User runs: bazel build //foo

         ┌──────────────┐
         │   Bazel CLI  │
         │   (client)   │
         └──────┬───────┘
                │
         [Connects to existing server OR starts new one]
                │
         ┌──────▼────────────────────────┐
         │   Bazel Server Process        │
         ├───────────────────────────────┤
         │ Cache:                        │
         │ - BUILD files (parsed)        │
         │ - Dependency graphs           │
         │ - Action results              │
         │ - Other metadata              │
         ├───────────────────────────────┤
         │ Execution:                    │
         │ - Action graph evaluation     │
         │ - Parallel execution          │
         │ - Output generation           │
         └───────────────────────────────┘
```

---

## Why Client-Server?

### Traditional Batch-Oriented Build (e.g., make)

```bash
$ make
# Process starts
# Loads everything from disk
# Executes build
# Process exits
# All state discarded

$ make                    # Next run: reload everything again!
```

**Problems:**
- ❌ Reload overhead on every invocation
- ❌ No persistent cache between builds
- ❌ Slower incremental builds

### Bazel's Server Approach

```bash
$ bazel build //foo
# Server starts (once)
# Caches BUILD files, graphs, etc.
# Executes build
# Server stays running

$ bazel build //bar    # Reuses same server!
# No reload overhead
# Caches shared between builds
# Fast queries on cached data
```

**Benefits:**
- ✅ **Persistent cache** — BUILD files, graphs, metadata cached across commands
- ✅ **Incremental efficiency** — Only reload changed files
- ✅ **Shared queries** — `bazel query` uses same cache as `bazel build`
- ✅ **Command sharing** — All clients to same workspace share same cache

---

## How It Works

### 1. Server Startup

When you run `bazel`, the client:

```
1. Checks if server is already running (using output base)
   └─ Output base: directory determined by workspace path + user ID

2. If server exists and correct version: connect to it

3. If server missing or wrong version:
   └─ Unpack Bazel archive
   └─ Start new server process
   └─ Confirm installation (verify mtimes unchanged)

4. Send command to server
5. Wait for results
6. Exit
```

### 2. Output Base

**Output Base** is a directory that uniquely identifies a Bazel workspace + user combination:

```bash
# Default location:
~/.cache/bazel/_bazel_<username>/<hash-of-workspace-path>

# Examples:
# Workspace: /home/alice/project1
# Output base: ~/.cache/bazel/_bazel_alice/7a3f2e1b/

# Workspace: /home/alice/project2
# Output base: ~/.cache/bazel/_bazel_alice/9c4d5a2x/

# Workspace: /home/bob/project1
# Output base: ~/.cache/bazel/_bazel_bob/7a3f2e1b/
# (different from Alice's due to different user)
```

**Why:** Multiple workspaces + multiple users can safely build simultaneously without conflicts.

### 3. Server Lifecycle

**Start:**
- Triggered by first `bazel` command
- Loaded from archive

**Running:**
- Accepts multiple sequential commands
- Maintains cache between commands
- Only one command executes at a time (queues additional invocations)

**Shutdown:**
- Automatically after idle timeout (3 hours, default)
  - Configurable: `--max_idle_secs=<seconds>`
  - Example: `bazel --max_idle_secs=600 build //...` (10 min timeout)

- Manually: `bazel shutdown`

**Restart:**
- Automatic if version mismatch detected
- Automatic if workspace lock held by different process

---

## Server Process Identification

When running `ps x` or `ps -e f`, Bazel servers appear as:

```bash
$ ps -e f
16143 ?  Sl     3:00 bazel(src-johndoe2) -server -Djava.library.path=...
```

**Format:** `bazel(<workspace-basename>)`

**Where:** `<workspace-basename>` is the directory enclosing your workspace root

**Example:**
```
Workspace: /home/alice/projects/src
Workspace root: /home/alice/projects/src/WORKSPACE
Enclosing dir: /home/alice/projects/

Server process name: bazel(projects)
```

**Use this to identify which workspace a process belongs to.**

---

## Concurrent Builds

### Same Workspace, Same User

Only **one command** executes at a time on a server. Concurrent invocations queue:

```bash
# Terminal 1
$ bazel build //very/large:target
# Starts immediately, server handles it

# Terminal 2 (same workspace)
$ bazel query //...
# BLOCKS waiting for build to finish
# Then executes query using same cache
```

**Advantage:** Queries automatically use cached build state

### Multiple Workspaces, Same User

Each workspace has its own server (different output bases):

```bash
# Terminal 1
$ cd /home/alice/project-a
$ bazel build //...
# Uses ~/.cache/bazel/_bazel_alice/project-a-hash/

# Terminal 2
$ cd /home/alice/project-b
$ bazel build //...
# Uses ~/.cache/bazel/_bazel_alice/project-b-hash/
# Different server, no conflict!
```

### Multiple Users, Same Machine

Each user has separate server (different user IDs in output base):

```bash
# Alice's shell
$ bazel build //...
# Uses ~/.cache/bazel/_bazel_alice/workspace-hash/

# Bob's shell (same machine, same workspace)
$ bazel build //...
# Uses ~/.cache/bazel/_bazel_bob/workspace-hash/
# Different servers, no conflict!
```

---

## Server Management in Scripts

When running automated builds in scripts, **manage server lifecycle explicitly**:

### Problem: Accumulating Idle Servers

```bash
# Script that builds many different directories
for dir in /data/project-*; do
  cd $dir
  bazel build //...
done

# Result: Multiple idle servers accumulating!
# Each project gets its own server, all stay resident
```

### Solutions

**1. Shutdown Explicitly**

```bash
for dir in /data/project-*; do
  cd $dir
  bazel build //...
  bazel shutdown    # Clean up before next project
done
```

**2. Set Short Idle Timeout**

```bash
bazel --max_idle_secs=60 build //...
# Server shuts down after 1 minute of inactivity
```

**3. Use Non-blocking Mode**

```bash
bazel --noblock_for_lock build //...
# Fails immediately if lock held, doesn't wait
# Useful for CI/CD (don't want to hang indefinitely)
```

### Output Base Isolation

For CI/CD with concurrent builds, use separate output bases:

```bash
# Job 1
bazel --output_base=/tmp/bazel-ci-job-1 build //...

# Job 2 (can run simultaneously, same machine)
bazel --output_base=/tmp/bazel-ci-job-2 build //...

# No locking conflicts, no cache interference
```

---

## Caching Benefits

### BUILD File Caching

```bash
$ bazel query //...
# First run: parses all BUILD files (~1-2 seconds)

$ bazel query //package:*
# Second run: uses cached BUILD files (instant!)
```

### Dependency Graph Caching

```bash
$ bazel build //app:main
# Builds dependency graph, caches it

$ bazel test //app:tests
# Reuses same dependency graph (fast)
```

### Action Result Caching

```bash
$ bazel build -c opt //...
# Compiles everything, caches results

$ bazel build -c fastbuild //...
# Different config, rebuilds only what's needed
```

---

## Version Management

**Important:** Client and server versions must match.

### Automatic Version Check

Every command, the client verifies:

```
1. Connect to existing server (if any)
2. Check server version matches client version
3. If mismatch detected:
   └─ Shut down old server
   └─ Start new server with correct version
4. Execute command
```

**Result:** Bazel version updates are seamless (automatic server restart).

---

## Troubleshooting

### Server Won't Start

**Error:** `bazel: command not found` or similar

**Causes:**
- Bazel archive corrupt
- Installation verification failed
- Permissions issue

**Solutions:**
```bash
bazel clean --expunge    # Remove entire server + cache
bazel build //...        # Restart from scratch

# Or explicitly kill and restart
bazel shutdown
bazel build //...
```

### Lock Contention

**Error:** `Error: could not acquire lock on <output-base>`

**Cause:** Multiple processes accessing same output base

**Solutions:**
```bash
# Use different output base
bazel --output_base=/tmp/bazel-build-1 build //...

# Or wait for existing build to finish
bazel build //...  # Blocks until server is free

# Or fail immediately (useful in CI)
bazel --noblock_for_lock build //... || echo "Build in progress"
```

### Orphaned Server

**Symptom:** High CPU usage, builds hang

**Cause:** Server process stuck or corrupt cache

**Solution:**
```bash
bazel shutdown     # Kill server
rm -rf <output-base>/  # Remove corrupted cache
bazel build //...  # Restart fresh
```

---

## Best Practices

### Development

```bash
# Accept reasonable idle timeout
bazel build //...

# Automatic shutdown after 3 hours of inactivity
# No manual management needed
```

### CI/CD

```bash
# Isolate each build job
bazel --output_base=/tmp/bazel-ci-$$  build //...

# No cross-job cache interference
# No locking between jobs
# Optional: set short timeout
bazel --max_idle_secs=300 build //...
```

### Scripting Multiple Projects

```bash
# Cleanup after each project
for dir in /data/project-*; do
  cd $dir
  bazel build //...
  bazel shutdown
done

# Or use isolated output bases
for i in $(seq 1 $NUM_PROJECTS); do
  cd /data/project-$i
  bazel --output_base=/tmp/bazel-$i build //...
done
```

### Monitoring Servers

```bash
# List all running Bazel servers
ps aux | grep "bazel.*-server"

# Identify which workspace each server belongs to
# (from process name: bazel(workspace-name))

# Kill specific server (if needed)
bazel shutdown  # In that workspace's directory
```

---

## See Also

- [[reference/calling-bazel-from-scripts]] — More details on scripting Bazel
- [[concepts/advanced/hermeticity]] — Why isolated processes matter
