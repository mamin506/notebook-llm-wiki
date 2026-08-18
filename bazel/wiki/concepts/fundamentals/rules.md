---
title: "Rules: The Core Building Block"
category: "concepts"
level: "fundamentals"
status: "growing"
sources: ["Rules.md"]
tags: ["rules", "fundamentals", "#recommended", "targets"]
related: ["[[concepts/fundamentals/targets]]", "[[concepts/fundamentals/build.bazel]]", "[[concepts/advanced/writing-custom-rules]]", "[[reference/general-rules]]"]
last_updated: "2026-07-29"
---

# Rules: The Core Building Block

A **rule** is the fundamental mechanism in Bazel for defining **how to build** something. It describes a series of **actions** that Bazel performs to transform inputs into outputs.

---

## What Is a Rule?

### Simple Definition

A rule specifies:
1. **Inputs** — Source files, dependencies, tools (everything needed to build)
2. **Actions** — Commands to run (e.g., "compile this C++ file with g++")
3. **Outputs** — Files produced by the actions
4. **Providers** — Information exposed to other rules that depend on this one

### Real-World Example: C++ Binary Rule

```python
# Example: compiling a C++ binary
cc_binary(
    name = "my_app",
    srcs = ["main.cc"],      # Inputs: source files
    deps = [":lib"],         # Dependencies: other targets
)

# What happens:
# 1. Bazel gathers all inputs (main.cc, lib outputs, compiler, standard lib)
# 2. Registers actions: run g++ to compile and link
# 3. Produces outputs: executable binary
# 4. Exposes providers: DefaultInfo (the binary), CcInfo (C++ metadata)
```

---

## Rule vs. Target

**Don't confuse these terms:**

- **Rule** — A *type* or *class* that defines the **how**. Example: `cc_binary` (a rule type)
- **Target** — An *instance* of a rule. Example: `//app:my_app` (a specific cc_binary target)

```
Rule:     cc_binary (the blueprint)
Target:   //app:my_app (an instance of cc_binary)
```

---

## Built-In vs. Custom Rules

### Built-In Rules (First-Class Rules)

Bazel comes with native rules for core functionality:

- `cc_binary`, `cc_library` — C/C++ compilation
- `java_binary`, `java_library` — Java compilation
- `py_binary`, `py_library` — Python packaging
- `genrule` — Generic shell command execution
- `filegroup` — Group files for distribution

These are implemented in Java/C++ inside Bazel itself and optimize execution.

### Custom Rules

You can define your own rules using **Starlark** (Bazel's extension language):

```python
# In a .bzl file
example_library = rule(
    implementation = _example_library_impl,
    attrs = {
        "srcs": attr.label_list(allow_files = [".example"]),
        "deps": attr.label_list(),
    },
)

# Then use in BUILD files:
example_library(
    name = "my_target",
    srcs = ["input.example"],
)
```

**Key insight:** All rules are conceptually equal. There's no distinction between built-in and custom rules—they follow the same architecture.

---

## Rule Anatomy

A rule consists of several key components:

### 1. Rule Definition (in .bzl file)

```python
my_rule = rule(
    implementation = _my_rule_impl,      # Function that builds it
    attrs = {                            # Attributes (parameters)
        "srcs": attr.label_list(),       # Source files
        "deps": attr.label_list(),       # Dependencies
        ...
    },
)
```

### 2. Attributes

**Attributes** are the rule's parameters. When you instantiate a rule in a BUILD file, you provide values for attributes:

```python
my_rule(
    name = "my_target",
    srcs = ["main.cc"],          # Attribute value
    deps = [":lib"],              # Attribute value
)
```

**Common attribute patterns:**
- `srcs` — Source files (input files to process)
- `deps` — Code dependencies (other targets this one depends on)
- `data` — Runtime data files (files needed at execution time)
- `hdrs` — Header files (for languages with headers)
- `outs` — Output files (declaring what outputs the rule produces)

### 3. Implementation Function

The implementation function is where the "magic" happens. It's a Python-like function that:
- Reads the target's attributes
- Reads dependencies via **providers**
- Registers **actions** (build commands)
- Returns **providers** to consumers

```python
def _my_rule_impl(ctx):
    # ctx is the rule context
    # ctx.attr.srcs = list of source files
    # ctx.attr.deps = list of dependencies
    
    # Declare output file
    output = ctx.actions.declare_file("output.txt")
    
    # Register action (e.g., run a command)
    ctx.actions.run_shell(
        command = "cat > $1",
        arguments = [output.path],
        outputs = [output],
    )
    
    # Return providers
    return [
        DefaultInfo(files = depset([output]))
    ]
```

---

## Three Phases of a Build

Understanding when rules execute is crucial:

### 1. Loading Phase
- Bazel reads `BUILD` files and `.bzl` files
- Instantiates rules (calls `my_rule(name="...", srcs=[...])`)
- **No rule logic runs yet; implementation functions are NOT called**

### 2. Analysis Phase
- Bazel calls each rule's **implementation function**
- Implementation functions **register actions** (but don't execute them)
- Creates a graph of actions
- **No external commands run; no files are read/written**

### 3. Execution Phase
- Bazel executes the action graph
- Actually runs compilers, linkers, etc.
- Writes output files
- **This is when actual building happens**

**Key takeaway:** Rule implementation functions cannot do I/O or run commands. They only describe *what* to do; execution happens later.

---

## Rules vs. Macros

**Rules** and **Macros** are often confused:

| Aspect | Rule | Macro |
|--------|------|-------|
| **Definition** | Starlark function that generates actions | Starlark function that calls rules |
| **When it runs** | Analysis phase (generates actions) | Loading phase (generates rule instances) |
| **What it returns** | Providers | Nothing (side effect: creates targets) |
| **Use case** | Logic, caching, dependency management | Convenience, reusable patterns |

```python
# Macro: a shortcut that generates multiple targets
def my_lib_macro(name, srcs):
    cc_library(name = name, srcs = srcs)
    cc_test(name = name + "_test", srcs = srcs)

# In BUILD file:
my_lib_macro(name = "mylib", srcs = ["lib.cc"])
# Expands to: cc_library(name="mylib", ...) + cc_test(name="mylib_test", ...)

# Rule: actual build logic
my_custom_rule(name = "mytarget", srcs = [...])  # Performs custom compilation
```

---

## Why Rules Matter

1. **Abstraction** — Hide complexity (users don't need to know compiler flags)
2. **Reusability** — Define once, use everywhere
3. **Composability** — Rules depend on other rules, building dependency graphs
4. **Caching** — Bazel caches rule outputs; identical inputs = reused outputs
5. **Distributed Builds** — Actions can run on remote machines

---

## See Also

- [[concepts/advanced/writing-custom-rules]] — Deep dive into implementing rules
- [[reference/general-rules]] — Built-in Bazel rules catalog
- [[concepts/fundamentals/targets]] — Instances of rules
- [[concepts/fundamentals/build.bazel]] — How to write BUILD files
