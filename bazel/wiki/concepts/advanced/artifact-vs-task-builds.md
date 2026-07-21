---
title: "Artifact-Based vs Task-Based Build Systems"
category: "concepts"
level: "advanced"
status: "seedling"
sources: ["Task-Based Build Systems.md", "Artifact-Based Build Systems.md"]
tags: ["architecture", "design-philosophy", "build-systems", "comparison"]
related: ["[[concepts/advanced/build-systems-landscape]]", "[[reference/bazel-vs-cmake]]", "[[concepts/advanced/hermeticity]]"]
last_updated: "2026-07-20"
---

# Artifact-Based vs Task-Based Build Systems

Understanding the fundamental difference between artifact-based and task-based build systems explains why Bazel is designed the way it is.

---

## The Paradigm Shift

### Task-Based (Traditional)

**Philosophy:** "Tell the build system HOW to build"

```
Engineer specifies: step 1 → step 2 → step 3 → output
System executes: step 1, then step 2, then step 3
```

**Examples:** Make, Maven, Gradle, Ant

### Artifact-Based (Modern)

**Philosophy:** "Tell the build system WHAT to build; let it figure out HOW"

```
Engineer specifies: "I need artifact X (which depends on Y and Z)"
System determines: build order, parallelization, caching, optimization
```

**Examples:** Bazel (Google's Blaze), Buck (Meta)

---

## Task-Based Build Systems: How They Work

### Definition

Tasks are scripts that can execute arbitrary logic. Each task specifies other tasks as dependencies.

### Example: Apache Ant

```xml
<project name="MyProject" default="dist">
    <target name="init">
        <mkdir dir="build"/>
    </target>
    
    <target name="compile" depends="init">
        <javac srcdir="src" destdir="build"/>
    </target>
    
    <target name="dist" depends="compile">
        <jar jarfile="dist/MyProject.jar" basedir="build"/>
    </target>
</project>
```

**Execution:** When user runs `ant dist`, Ant executes:
```
1. init (create directory)
2. compile (run javac)
3. dist (create JAR)
```

### Advantages

✅ **Powerful** — Tasks can do anything  
✅ **Flexible** — Support any build process  
✅ **Familiar** — Similar to shell scripts  
✅ **Extensible** — Easy to add custom logic  

---

## The Dark Side of Task-Based Systems

As projects scale, task-based systems hit fundamental limits:

### 1. Parallelization Difficulty

**Problem:** System doesn't know what tasks do, so can't safely parallelize.

```
Task A depends on B and C
B and C have no dependencies on each other
Safe to run in parallel? MAYBE.
- Maybe they both write to the same file → conflict
- Maybe they share resources → race condition
- System can't know → must run sequentially

Result: Wasted CPU cores
```

### 2. Incremental Build Failures

**Problem:** Detecting when to rebuild is implicit, not explicit.

```
Task: "Compile all Java files"
Question: Did this task already run?
Answer: "I don't know—maybe source changed, maybe a dep changed, 
         maybe an external file was downloaded, maybe a timestamp changed..."

Solution: Re-run every task (clean builds)
Result: No incremental build optimization
```

### 3. Maintenance Nightmare

**Common bugs:**
```
Bug 1: Task A depends on Task B's output
       Task B owner changes output location
       → Task A breaks (silent until someone runs it)

Bug 2: Task A depends on Task B → Task C
       Task C produces file needed by Task A
       Task B owner removes dependency on C
       → Task A fails (transitive dependency broken!)

Bug 3: Task writes to /tmp/shared_state
       Two teams run tasks in parallel
       → Race condition, non-deterministic results

Bug 4: Task A depends on "latest version"
       → Different results on different days
       → Build is non-reproducible
```

### 4. Script Maintenance Burden

As projects grow, build scripts become complex, fragile code—and no one wants to maintain them.

---

## Artifact-Based Build Systems: A Better Model

### Core Idea

Instead of tasks (imperative "do this"), use artifacts (declarative "I need this").

### Example: Bazel

```starlark
java_library(
    name = "mylib",
    srcs = ["MyLibrary.java"],
    deps = [":helper"],
)

java_binary(
    name = "myapp",
    srcs = ["Main.java"],
    deps = [":mylib"],
)
```

**What this says:**
```
Artifact "mylib" = compile MyLibrary.java
Artifact "myapp" = compile Main.java + link mylib
```

**What the build system determines:**
- Build order (helper → mylib → myapp)
- Parallelization (can build MyLibrary.java and helper independently)
- Caching (if inputs unchanged, reuse outputs)
- Incremental rebuilding (if only Main.java changed, recompile myapp but reuse mylib)

### The Functional Programming Analogy

Artifact-based systems resemble functional programming:

**Imperative (Task-Based):**
```python
# "Do these steps in order"
step1()  # Create directory
step2()  # Run compiler
step3()  # Create archive
```

**Functional (Artifact-Based):**
```
# "Declare what you want"
archive = jar(compile(sources))
# System figures out execution order
```

In functional programming, the compiler can:
- Parallelize (no side effects)
- Optimize (no state changes)
- Cache results (pure functions)

Same benefits for artifact-based builds!

---

## How Artifact-Based Solves Task-Based Problems

### Problem 1: Parallelization

**Task-Based:** "Don't know what tasks do, must run sequentially"

**Artifact-Based:** "Know exactly: this is a Java compiler + these inputs = this output"
```
Can safely parallelize:
- compile helper.java (inputs: helper.java)
- compile main.java (inputs: main.java)
→ No shared resources, safe to run in parallel ✅
```

### Problem 2: Incremental Builds

**Task-Based:** "Don't know what changed, rebuild everything"

**Artifact-Based:** "Artifact depends on: source files + compiler + flags"
```
If main.java changed but helper.java didn't:
- Recompile main.java (input changed)
- Reuse compiled helper.java (input unchanged) ✅
```

### Problem 3: Maintenance

**Task-Based:** "Tasks can do anything, no way to validate"

**Artifact-Based:** "Rules declare inputs/outputs, system validates"
```
java_library must:
- Take source files as input
- Produce a library as output
- Link dependencies

If rule breaks this contract → build fails with clear error ✅
```

### Problem 4: Reproducibility

**Task-Based:** "Scripts download 'latest', use timestamps, env vars"
```
Build Tuesday: get version 1.0
Build Wednesday: version updated to 1.1
→ Different results, no way to reproduce Tuesday's build
```

**Artifact-Based:** "All inputs must be declared, versioned, and hashed"
```
Build Tuesday: declare openssl 3.1.0 → hash ABC123
Build Wednesday: same declaration → same hash → same binary ✅
Checkout old branch → old dependency version → same binary ✅
```

---

## Key Differences Summary

| Aspect | Task-Based | Artifact-Based |
|--------|-----------|-----------------|
| **Unit of work** | Task (script) | Artifact (output) |
| **Engineer specifies** | HOW to build | WHAT to build |
| **Parallelization** | Difficult (implicit deps) | Natural (explicit outputs) |
| **Incremental build** | Must rebuild everything | Rebuild only changed parts |
| **Reproducibility** | Non-deterministic | Deterministic by design |
| **Caching** | Manual, unreliable | Automatic, content-based |
| **Scalability** | Poor at scale | Excellent at scale |
| **Learning curve** | Easy initially | Requires new thinking |

---

## The Trade-off

### Artifact-Based Costs

❌ **Reduced flexibility** — Can't do arbitrary things  
❌ **Declarative mode** — Different mental model than imperative scripts  
❌ **Strict rules** — Must follow framework (can't hack)  

### Artifact-Based Benefits

✅ **Safe parallelization** — System handles scheduling  
✅ **Fast incremental builds** — Only rebuild what changed  
✅ **Reproducibility** — Same source → same binary always  
✅ **Scalability** — Works at Google-scale projects  
✅ **Reliability** — No hidden bugs from scripts  

**The verdict:** For small projects, task-based is fine. For teams and scaling, artifact-based wins decisively.

---

## Why Bazel Chose Artifact-Based

Google built Blaze (Bazel's predecessor) as artifact-based because:

1. **Scale** — Google's codebase: billions of lines, millions of builds/day
2. **Consistency** — Needed reproducible builds across 100k+ engineers
3. **Performance** — Task-based couldn't keep up (parallelization critical)
4. **Correctness** — Implicit dependencies caused too many bugs

For smaller projects using task-based (Maven, Gradle) is reasonable. But for monorepos and large organizations, artifact-based is essential.

---

## See Also

- [[concepts/advanced/build-systems-landscape]] — Overview of all build systems
- [[reference/bazel-vs-cmake]] — Bazel vs CMake (task-based)
- [[concepts/advanced/hermeticity]] — Reproducibility in artifact-based systems
