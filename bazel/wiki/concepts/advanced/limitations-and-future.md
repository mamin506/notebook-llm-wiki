---
title: "Bazel's Limitations and Future Directions"
category: "concepts"
level: "advanced"
status: "seedling"
sources: []
tags: ["architecture", "limitations", "scalability", "future", "engineering-practice"]
related: ["[[concepts/advanced/bazel-design-decisions]]", "[[concepts/advanced/artifact-vs-task-builds]]", "[[concepts/advanced/build-systems-landscape]]"]
last_updated: "2026-07-20"
---

# Bazel's Limitations and Future Directions

Understanding Bazel's constraints illuminates where the next generation of build systems might go.

---

## The Fundamental Tension

Bazel solved **scale** by imposing **constraints**:

```
Flexibility ←────────────────→ Scalability

Task-based:                      Bazel:
├─ Do anything                   ├─ Declare what you need
├─ Maximum flexibility           ├─ Maximum scalability
└─ Breaks at scale              └─ Restrictive for small projects
```

But **constraints** create problems too.

---

## Known Limitations

### 1. Learning Curve: Declarative vs Imperative

**Problem:** Bazel requires engineers to think in declarative terms.

```
Traditional (shell, Make):
"Do this, then that, then that"
→ Familiar, intuitive

Bazel:
"I need this artifact, which depends on these"
→ Different mental model, steeper curve
```

**Who suffers:** Newcomers, engineers from dynamic language backgrounds, those used to shell scripting.

**Why it matters:** High onboarding cost delays adoption.

**Possible solutions:**
- Better tooling (more auto-generation)
- Better education (more tutorials)
- Hybrid approaches (mix declarative + imperative)

### 2. Configuration Verbosity

**Problem:** Explicitly declaring all dependencies means more configuration code.

```
Task-based:
├─ Implicit transitive: B depends on C automatically
└─ Less verbose (if it works)

Bazel:
├─ Explicit: must list B and C separately if both needed
└─ More verbose (but clearer)
```

**Who suffers:** Teams maintaining large numbers of BUILD files.

**Why it matters:** More code to maintain = more bugs possible.

**Possible solutions:**
- Auto-generation tools (`gazelle`, `buildozer`)
- IDE integration (auto-suggest missing deps)
- Macro systems (reduce duplication)

### 3. Windows Support Weakness

**Problem:** Bazel's sandboxing relies on Linux namespaces and macOS sandbox-exec. Windows support is weaker.

```
Linux:   Native sandboxing via namespaces → fast, secure
macOS:   Native sandboxing via sandbox-exec → fast, secure
Windows: Limited sandboxing options → slower, less secure
```

**Who suffers:** Windows developers.

**Why it matters:** Limits adoption in Windows-heavy organizations.

**Current state:** Improving, but not equivalent to Linux/macOS.

### 4. Incremental Build Ceiling

**Problem:** No matter how optimized, you still need to recompile when sources change.

```
Bazel's parallel execution:
├─ Optimizes: "how to compile efficiently"
├─ Can't optimize: "whether to compile"
└─ When code changes, must recompile
```

**Example:**
```
100,000 targets, 10,000 change
→ Must rebuild 10,000 (parallel helps, but still slow)
```

**Who suffers:** Large-monorepo developers during heavy refactors.

**Why it matters:** Build times still matter at extreme scale.

**Possible solutions:**
- More aggressive caching strategies
- Incremental compilation (language support)
- Remote execution (parallelize across workers)

### 5. Memory Usage at Scale

**Problem:** Bazel loads the entire dependency graph into memory.

```
Small project: 1,000 targets → few MB
Large monorepo: 1,000,000 targets → gigabytes
```

**Who suffers:** Large-monorepo developers on memory-constrained machines.

**Why it matters:** Some organizations have monorepos too large to load on normal developer machines.

**Possible solutions:**
- Lazy loading (load only relevant subgraphs)
- Streaming builds (don't load entire graph)
- Distributed analysis (analyze on remote server)

### 6. Strictness Can Be Oppressive

**Problem:** "Correctness" sometimes feels like "inflexibility."

```
Legitimate use case: "I want to do something non-standard"
Bazel's answer: "That violates our constraints"
Result: Frustration
```

**Examples:**
- Accessing system libraries (non-hermetic but sometimes necessary)
- Running commands with side effects
- Complex build logic that doesn't fit the model

**Who suffers:** Engineers working on non-standard projects.

**Why it matters:** Not all projects fit Bazel's model perfectly.

**Possible solutions:**
- More escape hatches (`genrule`, custom rules)
- Better documentation on when to NOT use Bazel
- Hybrid approaches

### 7. Dependency Hell Still Exists

**Problem:** Despite all constraints, diamond dependency problems can still occur.

```
A depends on C v1.0
B depends on C v2.0
A and B both needed in same binary
→ Conflict!
```

**Bazel's solution:** One-Version Rule (all code must use same version).

**But:** This sometimes requires significant refactoring.

**Who suffers:** Teams integrating multiple external dependencies.

**Why it matters:** Version conflicts still require manual resolution.

---

## Performance Ceilings

### Cache Effectiveness Limits

```
Best case (all cached):      0.1 seconds (lookup only)
Cold build (large mono):     10-30 minutes (compile everything)
Partial change:              1-5 minutes (recompile changed parts)
```

**Plateau:** You can't make compilation faster than the speed of compilation itself.

### Parallelization Limits

```
Ideal (infinite workers):    1 minute (critical path only)
Actual (limited workers):    5-10 minutes (bound by network + workers)
```

**Plateau:** Network latency and critical path dependencies limit speedup.

### Sandbox Overhead Limits

```
Per-action overhead:         ~20ms (sandbox creation)
10,000 actions × 20ms:       200 seconds overhead
```

**Plateau:** As actions get smaller, overhead becomes significant.

---

## Scenarios Where Bazel Struggles

### 1. Very Small Projects
- Overhead not worth the benefit
- **Better choice:** Makefile, shell scripts

### 2. Dynamic Languages (interpreted)
- Bazel designed for compiled languages
- Python/JavaScript often don't need build systems at all
- **Better choice:** Language-native package managers

### 3. Highly Custom Build Logic
- Bazel's constraint model doesn't fit
- Non-standard workflows don't map to artifact-based model
- **Better choice:** Custom scripts or specialized tools

### 4. Windows Development
- Weaker sandbox support
- Different ecosystem (Visual Studio, NuGet)
- **Better choice:** MSBuild, CMake (on Windows)

### 5. Real-Time Systems
- Bazel's hermeticity adds overhead
- Some embedded systems need precise control
- **Better choice:** Custom build systems

---

## Possible Future Directions

### 1. Better Incrementalism

**Goal:** Reduce time from "code change" to "ready to test"

**Approaches:**
- True incremental compilation (per-function, not per-file)
- Distributed incremental analysis (don't reanalyze whole graph)
- Machine learning to predict which targets to rebuild

### 2. Hybrid Declarative/Imperative

**Goal:** Get flexibility of scripts with consistency of declarations

**Approach:**
```
Declarative core (dependencies, parallelization)
+
Imperative extensions (for complex logic)
=
Best of both worlds?
```

### 3. Language-Specific Optimization

**Goal:** Optimize for each language's strengths

**Example:**
- Python: lightweight packages, no compilation
- C++: aggressive incremental compilation
- Java: persistent incremental compilation

### 4. Distributed-First Architecture

**Goal:** Assume remote execution from the start

**Benefits:**
- Removes local resource constraints
- Enables massive parallelism
- Caching works globally

**Challenge:** Network reliability becomes critical

### 5. Build System as Platform

**Goal:** Bazel as infrastructure for other tools

**Idea:**
- IDE integration (IntelliJ, VSCode)
- Language servers using Bazel's dependency info
- Testing frameworks using Bazel's execution model
- Profiling tools using Bazel's execution traces

---

## The Inevitable Tradeoff

Every improvement in one area creates tradeoffs elsewhere:

```
More incrementalism    → More complexity
Better IDE integration → More overhead
True flexibility       → Loss of guarantees
Faster builds          → Less reproducibility
```

**Lesson:** No build system is universally best. The question is: *for your constraints, what matters most?*

---

## When NOT to Use Bazel

**Choose Bazel if:**
- ✅ Large monorepo (100+ targets)
- ✅ Multiple languages
- ✅ Reproducibility critical
- ✅ Team size growing
- ✅ CI/CD automation important

**Choose something else if:**
- ❌ Small project (< 50 targets)
- ❌ Simple single-language
- ✅ Flexibility more important than consistency
- ✅ Your ecosystem has a standard tool (Maven for Java, npm for JavaScript)
- ✅ Build speed not critical

---

## The Deeper Insight

Your observation:

> "When a bottom-layer engineering practice thoroughly masters a domain, it dominates that field"

Bazel mastered **large-scale deterministic builds**. But:

1. **It can't master everything:** Other priorities (flexibility, ease of use) are secondary
2. **Its mastery has limits:** Windows, dynamic languages, real-time systems don't fit the model
3. **The next system** will master something else: perhaps "human productivity" or "real-time feedback"

---

## The Future

Rather than Bazel becoming the universal build system, we might see:

```
Domain 1: Large monorepos (100k+ LOC)
└─ Bazel

Domain 2: Microservices (small, independent projects)
└─ Language-native tools (npm, cargo, go mod)

Domain 3: Data science (notebooks, experiments)
└─ Jupyter, DVC, or custom orchestration

Domain 4: Machine learning (training pipelines)
└─ Specialized tools (Airflow, Kubeflow)

Domain 5: Embedded systems (resource-constrained)
└─ Custom, minimal build systems
```

Each domain will have its own "dominant" system that mastered its specific problem.

---

## See Also

- [[concepts/advanced/bazel-design-decisions]] — Why Bazel made its tradeoffs
- [[concepts/advanced/artifact-vs-task-builds]] — The architectural choice
- [[concepts/advanced/build-systems-landscape]] — How Bazel compares to alternatives
