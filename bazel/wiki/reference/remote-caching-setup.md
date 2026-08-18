---
title: "Remote Caching Setup and Configuration"
category: "reference"
level: "intermediate"
status: "growing"
sources: ["Remote Caching.md"]
tags: ["#remote-caching", "#cache", "#ci-cd", "#performance", "#distributed-builds"]
related: ["[[reference/remote-execution]]", "[[reference/execution-tags-and-caching]]", "[[patterns/building-for-production]]"]
last_updated: "2026-07-28"
---

# Remote Caching Setup and Configuration

Set up and configure remote caching to share build outputs across your team and CI system.

---

## Overview

A **remote cache** stores build outputs from actions so they can be reused across machines and builds. This can dramatically speed up builds—when outputs are already cached, Bazel skips compilation and downloads them instead.

```
Without remote cache:
  Engineer A builds: 5 minutes (local compilation)
  Engineer B builds: 5 minutes (local compilation)
  Total: 10 minutes of wasted CPU

With remote cache:
  Engineer A builds: 5 minutes (local compilation)
  Engineer B builds: 30 seconds (downloads from cache)
  Total: 5.5 minutes (45% faster)
```

---

## How Remote Caching Works

### The Build Process With Remote Cache

1. Bazel creates the dependency graph and action list
2. Bazel checks **local disk cache** for existing outputs
3. Bazel checks **remote cache** for existing outputs (cache hit)
4. For missing outputs, Bazel executes actions locally
5. New outputs are uploaded to remote cache
6. Future builds download from cache instead of recompiling

### Cache Data Structure

Remote cache stores two types of data:

```
Action Cache (AC):
  ├─ Maps action hash to result metadata
  └─ Stores stdout/stderr for each action

Content-Addressable Store (CAS):
  └─ Stores output files by content hash
```

---

## Backend Options

### 1. Google Cloud Storage (Recommended for teams)

**Best for:** Teams with Google Cloud infrastructure, managed service preference.

**Setup:**

```bash
# 1. Create storage bucket
gsutil mb -l us-central1 gs://my-bazel-cache

# 2. Create service account
gcloud iam service-accounts create bazel-cache \
  --display-name="Bazel Cache"

# 3. Grant permissions
gsutil iam ch serviceAccount:bazel-cache@PROJECT_ID.iam.gserviceaccount.com:objectCreator gs://my-bazel-cache

# 4. Create JSON key
gcloud iam service-accounts keys create cache-key.json \
  --iam-account=bazel-cache@PROJECT_ID.iam.gserviceaccount.com
```

**Configure in .bazelrc:**

```bash
build --remote_cache=https://storage.googleapis.com/my-bazel-cache
build --google_credentials=$HOME/cache-key.json
```

**Cost:** Pay per storage GB + egress bandwidth.

---

### 2. bazel-remote (Self-Hosted, Recommended)

**Best for:** Teams wanting full control, privacy, or air-gapped builds.

**Setup with Docker:**

```bash
# Pull image
docker pull buchgr/bazel-remote-cache

# Run cache server
docker run -d \
  -p 9092:9092 \
  -v /path/to/cache:/data \
  buchgr/bazel-remote-cache:latest \
    -dir /data \
    -max_size 100
```

**Configure in .bazelrc:**

```bash
build --remote_cache=http://cache-server:9092
```

**Features:**
- ✅ gRPC and REST APIs
- ✅ Garbage collection (clean old outputs)
- ✅ No external dependencies
- ✅ Open source

**Cost:** Hardware + bandwidth (you maintain).

---

### 3. nginx (Lightweight, Using WebDAV)

**Best for:** Simple setups with existing infrastructure.

**nginx.conf:**

```nginx
server {
    listen 8080;
    
    location /cache/ {
        root /path/to/cache/dir;
        dav_methods PUT;
        create_full_put_path on;
        client_max_body_size 1G;
        allow all;
    }
}
```

**Configure in .bazelrc:**

```bash
build --remote_cache=http://cache-server:8080/cache
```

**Pros:** Lightweight, uses standard HTTP.  
**Cons:** No garbage collection, limited features.

---

### 4. Other Options

**AWS S3:**
- Use S3 compatible HTTP API
- Supports versioning and lifecycle policies
- Good for AWS-native organizations

**Hazelcast, Apache httpd, Buildbarn, BuildGrid, NativeLink:**
- All support Bazel's HTTP caching protocol
- Choose based on your infrastructure

---

## Authentication

### HTTP Basic Auth

```bash
build --remote_cache=https://username:password@cache.example.com:443
```

**Warning:** Username/password transmitted in plaintext. Always use HTTPS.

### Google Cloud Authentication

```bash
# Service account key
build --google_credentials=/path/to/key.json

# Application default credentials
build --google_default_credentials
```

### Other Services

Refer to your cache provider's documentation for authentication setup.

---

## Configuration in .bazelrc

### Read and Write to Remote Cache

```bash
build --remote_cache=http://your-cache.example.com:9092
```

This:
- ✅ Reads from cache (use existing outputs)
- ✅ Writes to cache (store new outputs)
- ✅ All developers benefit from each other's builds

### Read-Only (Don't Write)

```bash
build --remote_cache=http://your-cache.example.com:9092
build --remote_upload_local_results=false
```

**Use case:** CI system builds the cache; developers only read.

### Exclude Targets from Remote Cache

Tag the target:

```python
java_library(
    name = "sensitive_lib",
    tags = ["no-remote-cache"],
    srcs = ["lib.java"],
)
```

Or use command line:

```bash
bazel build --remote_upload_local_results=false //my_lib
```

---

## Local Disk Cache

Use local disk as a cache layer (faster than remote):

```bash
build --disk_cache=$HOME/.bazel/cache
```

Or default location:

```bash
build --disk_cache
```

### Garbage Collection (Bazel 7.4+)

Automatically clean old cache entries:

```bash
build --disk_cache=$HOME/.bazel/cache
build --experimental_disk_cache_gc_max_size=50GB
build --experimental_disk_cache_gc_max_age=30d
build --experimental_disk_cache_gc_idle_delay=5m
```

---

## Caching Workflow Example

**Developer machine `.bazelrc`:**

```bash
# Use both local disk cache + remote cache
build --disk_cache=$HOME/.bazel/cache
build --remote_cache=https://cache.mycompany.com
build --google_credentials=$HOME/cache-key.json
```

**First build (nothing cached):**

```bash
$ bazel build //app:app
[1,234 actions, 500 local]
Total time: 5m 30s
```

**Second build (from cache):**

```bash
$ bazel build //app:app
[1,234 actions, 200 cache hits, 50 local]
Total time: 30s
```

---

## Unix Sockets

Connect via Unix domain socket (Linux):

```bash
build --remote_cache=http://localhost:9092
build --remote_proxy=unix:/var/run/bazel-cache.sock
```

**Use case:** Cache server on same machine, avoid network overhead.

---

## Cache Hit Optimization

### Why You Don't Get 100% Cache Hits

```
Action hash = SHA256(
  target name +
  source file hashes +
  compiler version +
  compilation flags +
  dependency outputs
)
```

**Cache miss when:**
- ❌ Source code changes (expected)
- ❌ Compiler version changes
- ❌ Compilation flags change
- ❌ Dependencies change
- ⚠️ Environment variables leak into action (keep .bazelrc clean)

### Improving Cache Hits

**1. Keep tool versions pinned:**

```bash
# Good: explicit version
cc_library(
    name = "mylib",
    toolchain = "@gcc-11//:toolchain",  # Pinned version
)

# Bad: implicit system compiler
cc_library(
    name = "mylib",
    # Uses whatever gcc is on PATH
)
```

**2. Whitelist environment variables:**

```bash
build --action_env=PATH=$PATH
build --action_env=CC=/path/to/gcc
```

Don't expose all environment; only what's needed.

**3. Use hermetic toolchains:**

```bash
# Toolchains declared as dependencies
bazel_dep(name = "rules_cc", version = "0.1.1")
```

---

## Troubleshooting

### Fewer Cache Hits Than Expected

**Check for:**

1. **Old `/etc/bazel.bazelrc` polluting cache**
   ```bash
   # Look for:
   cat /etc/bazel.bazelrc
   
   # It may export PATH or other env vars
   # Solution: Remove or update it
   ```

2. **Different compiler versions**
   ```bash
   # Verify all developers use same toolchain
   bazel query "@gcc//:compiler" --output=build
   ```

3. **Floating version dependencies**
   ```bash
   # Bad: "latest" version
   bazel_dep(name = "rules_cc", version = "latest")
   
   # Good: pinned version
   bazel_dep(name = "rules_cc", version = "0.1.1")
   ```

### Actions Not Being Cached

Check for `no-cache` tag:

```bash
# Find targets with no-cache
bazel query 'attr("tags", "no-cache", //...)'
```

### Remote Cache Unreachable

```bash
# Test connection
curl -I http://cache.example.com:9092

# Check Bazel logs
bazel build //app --verbose_failures
```

---

## Known Issues

### Input File Modification During Build

If source files change while Bazel builds, invalid results might be cached.

**Fix:** Enable change detection (Bazel 0.25.0+):

```bash
build --experimental_guard_against_concurrent_changes
```

### Tools Outside Workspace Not Tracked

```
Problem: Compiler at /usr/bin/gcc changes
Result: Two machines with different compilers share cache hits
        → Wrong outputs used

Solution: Use hermetic toolchains (declare compiler as dependency)
```

### Docker Container In-Memory State Lost

Running Bazel in Docker resets in-memory state on each invocation.

```bash
# This is inefficient in containers:
docker run myimage bazel build //app
docker run myimage bazel test //app

# Better: Run Bazel server persistently
docker run -d --name bazel-server myimage bazel server
docker exec bazel-server bazel build //app
docker exec bazel-server bazel test //app
```

---

## Performance Impact

**Typical improvements:**

| Scenario | Before Cache | With Cache | Speedup |
|----------|--------------|-----------|---------|
| First build (clean) | 5m | 5m | None |
| Incremental (1 file) | 2m | 30s | 4x |
| Parallel team builds | 5m × 5 people | 5m + (30s × 4 people) | 3.5x |
| CI builds | 10m | 2m | 5x |

**Factors:**
- Network bandwidth to cache server
- Size of cache hits
- Number of cache hits

---

## Migration Path

**Phase 1: Read-only cache**
```bash
build --remote_cache=http://cache.example.com
build --remote_upload_local_results=false
```

**Phase 2: Full cache (all contributors)**
```bash
build --remote_cache=http://cache.example.com
# Everyone can read and write
```

**Phase 3: CI-only writes**
```bash
# Developer machines: read-only
build --remote_cache=http://cache.example.com
build --remote_upload_local_results=false

# CI machines: read and write
build --remote_cache=http://cache.example.com
```

---

## See Also

- [[reference/execution-tags-and-caching]] — Control what gets cached with tags
- [[reference/remote-execution]] — Beyond caching: distributed execution
- [[reference/distributed-builds]] — Architecture of caching and RBE
