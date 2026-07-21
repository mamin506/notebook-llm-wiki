---
title: "Distributed Builds: Scaling Beyond Local Machines"
category: "reference"
level: "advanced"
status: "seedling"
sources: ["Distributed Builds.md"]
tags: ["distributed-builds", "remote-execution", "remote-cache", "scaling", "rbe"]
related: ["[[reference/remote-execution]]", "[[concepts/advanced/artifact-vs-task-builds]]", "[[concepts/advanced/hermeticity]]"]
last_updated: "2026-07-20"
---

# Distributed Builds: Scaling Beyond Local Machines

As codebases grow, local machines can't complete builds in reasonable time. Distributed builds spread work across multiple machines.

---

## The Scale Problem

As projects grow:

```
Codebase size: 100K lines → 1M lines → 100M lines
Build targets: 100 → 10K → 100K
Dependencies: 10 levels deep → 50 levels → 100+ levels
```

**Physics constraint:** No single machine can build fast enough.

**Solution:** Distribute work across machines.

```
Local machine (1 CPU core)
┌──────────────────────────────────┐
│ Build 100K targets sequentially │
│ Time: 10 hours                   │
└──────────────────────────────────┘

Distributed (100 workers)
┌─────────────────────────────────────┐
│ Each worker builds 1K targets       │
│ Parallel execution across 100 cores │
│ Time: ~10 minutes                  │
└─────────────────────────────────────┘
```

---

## Two Levels of Distribution

### Level 1: Remote Caching

**Idea:** Share build results across machines.

```
Engineer 1: Builds //lib:common
           Uploads result to remote cache

Engineer 2: Needs //lib:common
           Downloads from cache instead of rebuilding
           Saves 5 minutes of build time!
```

**Architecture:**
```
Developer A           Developer B
  │                     │
  ├─→ Remote Cache ←────┤
       (Redis/GCS)
       
"Did anyone else build this?"
"Yes, download it"
```

**Benefits:**
- ✅ Low-level dependencies shared
- ✅ Developers build faster
- ✅ Same hardware requirements (local CPU still needed)

**Requirements:**
- Builds must be 100% reproducible
- Cache key = target + hash of all inputs
- Download must be faster than building

### Level 2: Remote Execution

**Idea:** Send build work to remote workers.

```
Engineer 1: "Build //huge:project with 10K targets"
Engineer 1's machine: Sends command to build farm

Build farm (100 workers):
  Worker 1: Compile src/lib/a.cc
  Worker 2: Compile src/lib/b.cc
  ...
  Worker 100: Compile src/app/main.cc
  
All happens in parallel
  
Engineer 1: Gets result (100x speedup)
```

**Architecture:**
```
              Build Coordinator
              (Scheduler/Master)
                    │
        ┌───────────┼───────────┐
        │           │           │
      Worker      Worker      Worker
        1           2           3
      
Engineer → Send job → Coordinator → Distribute to workers
          ↓ Get result ←─────────────┘
```

**Benefits:**
- ✅ Near-infinite parallelism (100+ workers)
- ✅ 10-100x speedup on large builds
- ✅ No local CPU/memory constraints

**Requirements:**
- Perfectly hermetic builds (no system dependencies)
- Deterministic outputs (same input → same binary)
- Works well with artifact-based systems

---

## How Remote Caching Works

### The Cache Lookup Flow

```
1. User runs: bazel build //myapp

2. Bazel computes action: "Compile main.cc"
   - Input file: main.cc
   - Compiler: gcc-11
   - Flags: -O2
   - → Hash all inputs: "ABC123"

3. Check remote cache:
   "Do you have ABC123?"
   
   Cache says: "Yes, I have it"
   → Download artifact (fast!)
   
   OR
   
   Cache says: "No"
   → Build locally
   → Upload result to cache
```

### Cache Key Formula

```
Cache key = SHA256(
    target name +
    source file hashes +
    compiler version +
    compilation flags +
    dependency outputs
)
```

**Why hashing matters:**
- Different inputs → different hash → no false hits
- Same inputs (anywhere, anytime) → same hash → can reuse

### Reproducibility Requirement

For remote caching to work, builds MUST be reproducible.

```
Reproducible:
  bazel build //app on Monday → binary A (hash XYZ)
  bazel build //app on Friday → binary A (hash XYZ)
  
  Download from cache on Friday? → Get binary A safely ✅

Non-reproducible:
  Build includes: timestamp, UUID, system-specific paths
  Monday: binary A (hash XYZ)
  Friday: binary B (hash different)
  
  Downloaded cached artifact ≠ locally built artifact ❌
```

---

## How Remote Execution Works

### The Build Master Flow

```
1. Engineer: "Build //myapp"
   └─→ Send to build coordinator

2. Build coordinator:
   - Parse dependency graph
   - Determine action order
   
3. Schedule on workers:
   Action: compile_a.cc
   Inputs: a.cc, stdlib headers, compiler
   → Send to Worker 1
   
   Action: compile_b.cc
   Inputs: b.cc, stdlib headers, compiler
   → Send to Worker 2
   
   (Run in parallel)

4. Workers execute:
   - Download inputs
   - Execute action
   - Upload outputs to cache
   
5. Final link step (after all compilation done):
   - Download all object files
   - Link locally or on worker
   - Return final binary
```

### Dependency Management in RBE

```
            Cache
            (Shared storage)
              ▲
              │
        ┌─────┼─────┐
        │     │     │
      Worker1 Worker2 Worker3
      
Action A (on Worker 1):
- Needs: input.txt + compiler
- Produces: object1.o
- Stores in cache

Action B (on Worker 2):
- Needs: object1.o (from Action A)
- Looks in cache: "Is object1.o ready?"
- Master blocks until Action A finishes
- Action B downloads object1.o from cache
- Continues
```

---

## Remote Caching vs Remote Execution

| Aspect | Remote Cache | Remote Execution |
|--------|--------------|------------------|
| **What's shared** | Build artifacts only | Entire build work |
| **Local CPU used** | Yes (full build) | No/minimal |
| **Speedup** | 2-5x (cache hits) | 10-100x (parallelism) |
| **Infrastructure** | Simple (storage) | Complex (many workers) |
| **Cold start** | Slow (first build) | Same (first time slow) |
| **Cost** | Low | High (many machines) |
| **When to use** | Teams, small orgs | Large monorepos, CI/CD |

---

## Google's Implementation: Blaze/Bazel

Google has run distributed builds since 2008 with two systems:

### Remote Cache: ObjFS

```
Backend: Bigtable (distributed database)
         Stores all build outputs

Frontend: objfsd (FUSE daemon)
          Runs on each developer machine
          Lazy-downloads only requested files
          
Result: Developers see build outputs as local files
        But only download what they actually use
        Reduces disk/network by 2-3x
```

### Remote Execution: Forge

```
Client: Distributor (in Bazel)
        Sends each action request

Server: Scheduler
        Maintains action result cache
        Returns immediately if already built
        Places new actions in queue
        
Workers: Executor pool
         Read from queue
         Execute actions
         Store results in ObjFS
         
Result: Millions of builds/day
        Achieves 2x speedup over local builds
        Enables CI/CD at massive scale
```

---

## Challenges in Distributed Builds

### 1. Hermeticity

**Problem:** Workers don't have your machine's setup.

**Solution:** All dependencies explicitly declared.

```
❌ Bad (non-hermetic):
cc_binary(
    name = "app",
    srcs = ["main.cc"],
    # Assumes /usr/lib/libssl.so exists
)

✅ Good (hermetic):
cc_binary(
    name = "app",
    srcs = ["main.cc"],
    deps = ["@openssl//:crypto"],  # Explicit!
)
```

### 2. Determinism

**Problem:** Different builds produce different binaries.

**Solution:** Remove non-deterministic operations.

```
❌ Non-deterministic:
- Include __DATE__ in binary
- Use random number as version
- Hash set iteration order
- System-specific paths

✅ Deterministic:
- Fixed version string
- Deterministic ordering
- Use deterministic data structures
- Relative paths only
```

### 3. Communication Overhead

**Problem:** Network latency adds up.

**Solution:** Batch actions, optimize network.

```
Send 1 action: 100ms network overhead + 500ms compile = 600ms
Send 100 actions: 100ms network overhead + 50,000ms compile = 50,100ms
(overhead negligible)
```

### 4. Fallback Strategy

**Problem:** If remote unavailable, what happens?

**Solution:** Automatic fallback to local.

```
Try remote execution
  ↓
If fails or too slow
  ↓
Fall back to local
  ↓
User still gets result (just slower)
```

---

## When to Use Distributed Builds

### Remote Caching: Almost Always

```
✅ Use if:
- Team of 2+ engineers
- Any shared dependencies
- CI/CD system
- Want faster feedback

❌ Skip if:
- Solo developer
- Tiny project
- Very fast builds already
```

### Remote Execution: Large Scale

```
✅ Use if:
- 10,000+ build targets
- CI/CD with many commits/day
- Monorepo with slow builds
- Can afford build farm infrastructure

❌ Skip if:
- < 1000 targets
- Build already fast
- No infrastructure budget
- Distributed system complexity unwanted
```

---

## Bazel's RBE Integration

Bazel natively supports both:

```bash
# Remote cache only
bazel build --remote_cache=grpcs://cache.example.com //myapp

# Remote execution
bazel build \
  --remote_executor=grpcs://executor.example.com \
  --remote_cache=grpcs://cache.example.com \
  //myapp
```

See [[reference/remote-execution]] for detailed setup.

---

## See Also

- [[reference/remote-execution]] — Detailed RBE setup and configuration
- [[concepts/advanced/artifact-vs-task-builds]] — Why artifact-based scales better
- [[concepts/advanced/hermeticity]] — Reproducibility requirements
