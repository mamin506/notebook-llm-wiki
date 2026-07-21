---
title: "Bazel Design Decisions: Tradeoffs and Philosophy"
category: "concepts"
level: "advanced"
status: "seedling"
sources: []
tags: ["architecture", "design", "philosophy", "engineering-practice", "tradeoffs"]
related: ["[[concepts/advanced/artifact-vs-task-builds]]", "[[concepts/advanced/hermeticity]]", "[[concepts/advanced/build-systems-landscape]]"]
last_updated: "2026-07-20"
---

# Bazel Design Decisions: Tradeoffs and Philosophy

Understanding *why* Bazel made certain choices reveals deeper engineering principles.

---

## Core Philosophy: Correctness Over Convenience

Bazel's fundamental tradeoff:

```
Convenient (Task-Based):        Correct (Artifact-Based):
├─ "Do whatever you want"       ├─ "Declare what you need"
├─ Flexible and powerful        ├─ Strict and constrained
├─ Easy to learn                ├─ Requires new thinking
├─ Hard to scale                └─ Scales to billions of LOC
└─ Non-deterministic
```

**The Choice:** Bazel chose **correctness at scale** over convenience for individuals.

**Why:** Google's scale (billions of lines, 100k+ engineers) made individual convenience irrelevant. The system *must* be reliable.

---

## Decision 1: Artifact-Based, Not Task-Based

### The Question
"Should build systems let engineers write arbitrary scripts (tasks) or constrain them to declare inputs/outputs (artifacts)?"

### Bazel's Answer
Artifact-based. Engineers declare **what** to build; Bazel determines **how**.

### Tradeoff

**Task-Based (e.g., Make, Maven, Gradle):**
- ✅ Flexible: do anything you want
- ✅ Powerful: no constraints
- ❌ Non-deterministic: same source → different binaries
- ❌ Non-parallelizable: system can't know what's safe to parallelize
- ❌ Scales poorly: build speeds degrade with codebase size

**Artifact-Based (Bazel):**
- ✅ Deterministic: same source → same binary always
- ✅ Parallelizable: system knows it's safe
- ✅ Scalable: works at Google scale
- ❌ Restrictive: can't do arbitrary things
- ❌ Learning curve: requires new mental model

### Why This Matters
At Google's scale, **non-determinism is catastrophic**:
- 100k engineers, millions of builds per day
- If "same source" produces different binaries on different machines, you can't trust anything
- CI/CD breaks, bisecting becomes impossible, debugging becomes nightmare

**Conclusion:** The cost of flexibility (non-determinism) exceeded its value at scale.

---

## Decision 2: Hermetic Builds Required

### The Question
"Should build systems trust the host environment (compiler, libraries, etc.) or require all dependencies explicitly?"

### Bazel's Answer
All dependencies must be explicit. No relying on system state.

### Tradeoff

**Environment-Dependent:**
- ✅ Easy to set up: "just install gcc"
- ✅ Familiar: traditional model
- ❌ Fragile: different machines → different results
- ❌ Non-reproducible: can't reproduce old builds if environment changed

**Hermetic (Bazel):**
- ✅ Reproducible: same source → same environment → same binary
- ✅ Scalable: works across team, across time, across machines
- ✅ Safe: no "it works on my machine" problems
- ❌ Overhead: must download/version all tools
- ❌ Complexity: requires careful dependency declaration

### Why This Matters
Non-hermetic builds are a **hidden cost**:

```
Traditional:
  Day 1: Developer A builds app, works fine
  Day 2: System admin updates OpenSSL
  Day 3: Developer A rebuilds, app broken
  Question: Was code change bad, or environment change bad?
  → Debugging nightmare

Hermetic (Bazel):
  Day 1: Developer A builds with OpenSSL 3.1.0
  Day 2: System admin updates OpenSSL
  Day 3: Developer A rebuilds with OpenSSL 3.1.0 (unchanged)
  Result: Identical behavior
```

**Conclusion:** The cost of hermeticity (explicit dependencies) is worth the certainty it provides.

---

## Decision 3: Strict Sandboxing by Default

### The Question
"Should each build action be isolated from others, or can they share the filesystem?"

### Bazel's Answer
Each action gets its own sandbox. No implicit sharing.

### Tradeoff

**Shared Filesystem:**
- ✅ Fast: no sandbox overhead
- ✅ Simple: actions can write anywhere
- ❌ Non-deterministic: race conditions, hidden dependencies
- ❌ Fragile: one bad action breaks everything

**Sandboxed (Bazel):**
- ✅ Safe: actions can't interfere
- ✅ Deterministic: no race conditions
- ✅ Catches bugs: reveals missing dependencies
- ❌ Slower: sandbox creation overhead (~10-50ms per action)
- ❌ Complex: requires kernel support (Linux namespaces, macOS sandbox)

### Why This Matters
Hidden dependencies cause **mysterious bugs**:

```
Action A: Compile foo.o
          Side effect: creates /tmp/log.txt (undeclared!)

Action B: Compile bar.o
          Uses /tmp/log.txt

Later, Action A gets cached, doesn't re-run
→ bar.o gets wrong log.txt
→ Intermittent build failures
→ Impossible to debug

With sandboxing:
Action A can't create /tmp/log.txt (not in its sandbox)
→ Error immediately
→ Easy to fix
```

**Conclusion:** The overhead of sandboxing catches errors that would be hidden otherwise.

---

## Decision 4: No Implicit Dependencies

### The Question
"Should transitive dependencies be automatic or explicit?"

### Bazel's Answer
Explicit. If A depends on B and B depends on C, A must explicitly depend on C if it uses C.

### Tradeoff

**Implicit (Traditional):**
- ✅ Less typing: don't repeat dependencies
- ✅ Convenient: automatically get transitive
- ❌ Fragile: if B removes C, A breaks silently
- ❌ Non-obvious: what does A actually depend on?

**Explicit (Bazel, "strict transitive deps"):**
- ✅ Clear: BUILD file shows exact dependencies
- ✅ Safe: B can remove C without breaking A (if A doesn't use it)
- ✅ Maintainable: dependencies don't accumulate
- ❌ More verbose: must list all deps
- ❌ Initial effort: refactoring existing codebases painful

### Why This Matters
Google spent **years refactoring** their codebase to enforce strict transitive dependencies.

```
Before: Dependency chains accumulated
        A depends on B, which depended on C, which depended on D, ...
        Removing D broke A (which never directly used D)
        Builds got slower and more fragile

After: Dependencies only what you need
       A depends only on what it actually uses
       B can change without affecting A
       Builds are faster and more stable
```

**Lesson:** Short-term pain (refactoring) for long-term gain (stability).

---

## Decision 5: Language-Agnostic Rules, Not Plugins

### The Question
"How should Bazel support new languages: plugin system or built-in rules?"

### Bazel's Answer
Language-agnostic rules (like `genrule`, `run`) plus language-specific rulesets in separate repositories (rules_python, rules_cc, etc).

**NOT** a plugin system.

### Tradeoff

**Plugin System:**
- ✅ Flexible: any language can extend Bazel
- ✅ Decentralized: language communities own their rules
- ❌ Fragmented: 10 different Python implementations
- ❌ Inconsistent: quality varies wildly
- ❌ Hard to optimize: Bazel can't understand what plugins do

**Rulesets in Repos:**
- ✅ Consistent: one canonical implementation per language
- ✅ Maintainable: community-driven but coordinated
- ✅ Optimizable: Bazel understands rule internals
- ✅ Testable: rules live in repositories with tests
- ❌ Less flexible: must follow Bazel's constraint model
- ❌ Slower to innovate: requires coordination

### Why This Matters
Consistency beats flexibility at scale:

```
Plugin approach:
├─ Team A writes Python rules (basic)
├─ Team B writes Python rules (advanced)
├─ Team C writes Python rules (Cython support)
└─ Chaos: which one do you use?

Ruleset approach:
└─ One rules_python maintained by community
   └─ Everyone uses same approach
   └─ Easy to understand and maintain
```

**Conclusion:** Sacrificing flexibility for consistency pays off.

---

## Decision 6: Declarative, Not Imperative Configuration

### The Question
"Should BUILD files be imperative (do this, then that) or declarative (here's what I want)?"

### Bazel's Answer
Declarative. You declare targets and dependencies; Bazel figures out execution.

### Tradeoff

**Imperative (task-based):**
- ✅ Familiar: looks like shell scripts
- ✅ Powerful: can express any logic
- ❌ Order-dependent: step 1 must precede step 2
- ❌ Non-parallelizable: system can't reorder steps
- ❌ Hard to analyze: system doesn't know what's happening

**Declarative (Bazel):**
- ✅ Parallelizable: Bazel optimizes execution order
- ✅ Analyzable: Bazel understands the structure
- ✅ Reproducible: same structure → same execution
- ❌ Restrictive: can't express arbitrary logic
- ❌ Different mental model: requires functional thinking

### Why This Matters
The power to parallelize is the power to scale:

```
Imperative: 1000 sequential steps
            → Must wait for step 1, then 2, then 3... then 1000
            → Takes 1000 * (step_time) = slow

Declarative: 1000 independent steps
             → Bazel runs 100 in parallel
             → Takes 10 * (step_time) = 100x faster
```

---

## Decision 7: Deterministic Version Selection (MVS)

### The Question
"How should Bazel resolve dependency versions when conflicts arise?"

### Bazel's Answer
Minimal Version Selection (MVS). Pick the minimum version that satisfies all constraints.

### Tradeoff

**Automatic Latest (e.g., Gradle):**
- ✅ Always get latest features
- ✅ Security patches automatic
- ❌ Non-deterministic: build changes over time
- ❌ Breaking changes: minor version update might break you

**Deterministic MVS (Bazel):**
- ✅ Deterministic: same source → same dependencies forever
- ✅ Explicit: you control version updates
- ✅ Safe: you test before upgrading
- ❌ Manual: must explicitly update versions
- ❌ Requires discipline: can't ignore security updates

### Why This Matters
Non-deterministic dependencies are a **security nightmare**:

```
Automatic latest:
  Day 1: Build works
  Day 2: Dependency gets auto-updated
  Day 3: Build broken by breaking change
  Day 4: Security incident from new version
  → Chaos

Deterministic MVS:
  Day 1: Build works
  Day 2: Dependency update available (you see it)
  Day 3: You test and evaluate
  Day 4: You explicitly update if safe
  → Control and safety
```

---

## Unifying Theme: Control and Understanding

All these decisions share a theme:

```
Task-Based/Flexible:
└─ "Let's not constrain engineers"
   └─ "They're smart, they'll handle it"
   └─ → Works until: scale, time, team changes

Artifact-Based/Strict:
└─ "Let's require engineers to declare intent"
   └─ "The system will handle the complexity"
   └─ → Works at: scale, time, team changes
```

**Bazel's bet:** *Understanding and explicit declaration scale; flexibility and implicitness don't.*

---

## The Deep Principle

Your earlier observation:

> "When a bottom-layer engineering practice thoroughly masters a domain, it dominates that field"

Bazel's design decisions reflect this principle:

- **Artifact-based:** Masters "what actually needs to be built"
- **Hermetic:** Masters "what the build actually depends on"
- **Sandboxed:** Masters "what actions can safely do in parallel"
- **Explicit deps:** Masters "what code actually uses"
- **Deterministic versions:** Masters "which dependencies are safe"

Each decision requires **deep understanding** of build systems. That understanding becomes Bazel's competitive advantage.

---

## Implications

### For Users
Understand the *why*, not just the *what*. When you understand why Bazel makes these choices, you:
- Accept constraints more gracefully
- Make better architectural decisions
- Contribute meaningfully to the project
- Anticipate Bazel's behavior

### For Engineers Building Systems
When designing a system:
1. Identify what you're trying to master (e.g., "deterministic builds")
2. Accept the constraints necessary to master it (e.g., "explicit dependencies")
3. Invest in tooling to reduce the friction (e.g., `buildozer` for auto-adding deps)
4. Scale: your mastery becomes your advantage

### For the Industry
The lesson: *Correctness requires constraints*. This applies beyond build systems:
- Languages with strict type systems scale better than dynamic ones (at scale)
- Immutable architectures prevent more bugs than mutable ones
- Explicit APIs work better than implicit magic

---

## See Also

- [[concepts/advanced/artifact-vs-task-builds]] — How these decisions enabled artifact-based architecture
- [[concepts/advanced/limitations-and-future]] — Where these tradeoffs break down
