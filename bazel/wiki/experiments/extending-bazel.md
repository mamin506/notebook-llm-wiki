---
title: "Extending Bazel: Custom Rules and Rulesets"
category: "experiments"
level: "advanced"
status: "seedling"
sources: []
tags: ["extending", "custom-rules", "rulesets", "advanced", "deep-dive"]
related: ["[[concepts/advanced/bazel-design-decisions]]", "[[concepts/advanced/build-systems-landscape]]", "[[experiments/publishing-bazel-rules]]"]
last_updated: "2026-07-20"
---

# Extending Bazel: Custom Rules and Rulesets

For engineers who want to understand and extend Bazel itself—not just use it.

---

## Why Extend Bazel?

**When to write custom rules:**

```
Bazel provides:           Custom rules needed when:
├─ cc_library, cc_binary  ├─ Compiling exotic language
├─ py_binary, py_library  ├─ Non-standard build process
├─ java_binary            ├─ Custom code generation
└─ genrule (escape hatch) └─ Integration with legacy tools
```

**Why it matters:**
Not every language/tool has a Bazel ruleset. Knowing how to extend Bazel is how you adapt it to your needs.

---

## Core Concepts: Rules in Bazel

### What Is a Rule?

A **rule** is a reusable build target that:

1. **Declares inputs** (srcs, deps, data)
2. **Declares outputs** (what artifact it produces)
3. **Specifies actions** (how to transform inputs → outputs)

```starlark
# Built-in rule
cc_binary(
    name = "myapp",
    srcs = ["main.cc"],
    deps = ["//lib:mylib"],
)

# What Bazel does internally:
# 1. Declares inputs: main.cc, mylib
# 2. Declares output: executable named "myapp"
# 3. Specifies actions: invoke C++ compiler with these inputs
```

### Rule Anatomy (Starlark)

```starlark
def my_custom_rule_impl(ctx):
    """
    Implementation function.
    
    Args:
        ctx: context object with access to:
            - ctx.files.srcs: source files
            - ctx.attr.deps: dependencies
            - ctx.attr.name: target name
            - ctx.actions: tools to create actions
    
    Returns:
        list of providers (outputs)
    """
    
    # 1. Get inputs
    input_file = ctx.files.srcs[0]
    
    # 2. Declare output
    output_file = ctx.actions.declare_file(
        "{}.processed".format(ctx.attr.name)
    )
    
    # 3. Create action (the work that produces output)
    ctx.actions.run(
        executable = ctx.executable.tool,
        arguments = [input_file.path, output_file.path],
        inputs = [input_file],
        outputs = [output_file],
    )
    
    # 4. Return providers (how other targets use this output)
    return [
        DefaultInfo(files = depset([output_file])),
    ]

# Rule definition
my_custom_rule = rule(
    implementation = my_custom_rule_impl,
    attrs = {
        "srcs": attr.label_list(allow_files = True),
        "tool": attr.label(
            executable = True,
            cfg = "exec",
        ),
    },
)
```

---

## Creating a Simple Custom Rule

### Example: Protobuf Code Generator

**Goal:** Create a rule that generates C++ code from .proto files.

**The rule:**

```starlark
# my_rules.bzl
def proto_gen_impl(ctx):
    """Generate C++ from protobuf files."""
    
    # Inputs
    proto_files = ctx.files.srcs  # The .proto files
    
    # Outputs
    cc_files = []
    h_files = []
    for proto in proto_files:
        base = proto.basename.replace(".proto", "")
        cc_files.append(
            ctx.actions.declare_file("{}.pb.cc".format(base))
        )
        h_files.append(
            ctx.actions.declare_file("{}.pb.h".format(base))
        )
    
    # Action: run protoc compiler
    ctx.actions.run(
        executable = ctx.executable.protoc,
        arguments = [
            "--cpp_out={}".format(
                ctx.actions.declare_directory("gen").path
            ),
        ] + [f.path for f in proto_files],
        inputs = proto_files,
        outputs = cc_files + h_files,
    )
    
    # Return outputs
    return [
        DefaultInfo(files = depset(cc_files + h_files)),
        # Custom provider: tell dependent rules about generated files
        OutputGroupInfo(
            cc_source = depset(cc_files),
            cc_header = depset(h_files),
        ),
    ]

proto_gen = rule(
    implementation = proto_gen_impl,
    attrs = {
        "srcs": attr.label_list(
            allow_files = [".proto"],
        ),
        "protoc": attr.label(
            executable = True,
            cfg = "exec",
        ),
    },
)
```

**Using the rule in BUILD.bazel:**

```starlark
load(":my_rules.bzl", "proto_gen")

proto_gen(
    name = "generated",
    srcs = ["messages.proto"],
    protoc = "//tools:protoc",  # Where's the compiler?
)

cc_library(
    name = "mylib",
    srcs = [":generated"],  # Uses generated C++
    hdrs = [":generated"],
)
```

---

## Creating a Ruleset (Multi-File)

For production rules, organize into a ruleset (separate repository):

```
my_ruleset/
├── MODULE.bazel
├── BUILD.bazel
├── defs.bzl          # Rule definitions
├── impl.bzl          # Rule implementations
├── providers.bzl     # Custom providers
└── examples/
    ├── BUILD.bazel
    └── example.proto
```

**my_ruleset/MODULE.bazel:**

```starlark
module(
    name = "my_protobuf_rules",
    version = "1.0.0",
)

bazel_dep(name = "rules_cc", version = "0.1.1")
```

**my_ruleset/defs.bzl:**

```starlark
"""Public API for protobuf rules."""

load(":impl.bzl", "proto_gen_impl")

proto_gen = rule(
    implementation = proto_gen_impl,
    # ... attrs ...
)
```

---

## Advanced: Providers and Information Flow

### The Provider Pattern

**Problem:** How does one rule tell another "here's what I produced"?

**Solution:** Providers—structured data that rules pass to dependents.

```starlark
def my_rule_impl(ctx):
    """Create custom provider."""
    
    # ... build something ...
    
    # Standard provider: what files did we produce?
    default_info = DefaultInfo(files = depset([output_file]))
    
    # Custom provider: structured metadata
    my_info = MyInfo(
        message = "Build succeeded",
        output = output_file,
        metadata = {"version": "1.0"},
    )
    
    return [default_info, my_info]

# Define custom provider
MyInfo = provider(
    doc = "Information about my rule output",
    fields = {
        "message": "Status message",
        "output": "Generated file",
        "metadata": "Build metadata",
    },
)
```

### Using Providers

```starlark
def consuming_rule_impl(ctx):
    """Use another rule's provider."""
    
    # Get the provider from a dependency
    dep = ctx.attr.dep
    my_info = dep[MyInfo]  # Access custom provider
    
    # Use the data
    print(my_info.message)  # "Build succeeded"
    input_file = my_info.output
    
    # ... do something with it ...
```

---

## Common Patterns

### Pattern 1: Wrapping External Tools

**Goal:** Integrate a tool (not a language) into Bazel.

```starlark
def image_optimizer_impl(ctx):
    """Compress images using external tool."""
    
    input_image = ctx.files.srcs[0]
    output_image = ctx.actions.declare_file(
        "optimized_{}".format(input_image.basename)
    )
    
    ctx.actions.run(
        executable = ctx.executable.optimizer_tool,
        arguments = [
            input_image.path,
            output_image.path,
            "--quality", ctx.attr.quality,
        ],
        inputs = [input_image],
        outputs = [output_image],
    )
    
    return [DefaultInfo(files = depset([output_image]))]

image_optimizer = rule(
    implementation = image_optimizer_impl,
    attrs = {
        "srcs": attr.label_list(allow_files = [".jpg", ".png"]),
        "optimizer_tool": attr.label(executable = True, cfg = "exec"),
        "quality": attr.string(default = "high"),
    },
)
```

### Pattern 2: Aggregating Multiple Outputs

**Goal:** Rule that produces multiple files for different purposes.

```starlark
def multi_output_rule_impl(ctx):
    """Produce different outputs for different uses."""
    
    outputs = []
    
    for variant in ["debug", "release", "profiling"]:
        out = ctx.actions.declare_file("app_{}".format(variant))
        ctx.actions.run(...)  # Build variant
        outputs.append(out)
    
    return [
        DefaultInfo(
            files = depset(outputs),
            executable = outputs[1],  # Release is the default
        ),
        OutputGroupInfo(
            debug = depset([outputs[0]]),
            release = depset([outputs[1]]),
            profiling = depset([outputs[2]]),
        ),
    ]
```

### Pattern 3: Aspect-Driven Processing

**Goal:** Apply a transformation to every target of a certain type.

```starlark
def code_coverage_aspect_impl(target, ctx):
    """Add coverage to every test."""
    
    if not hasattr(target, "executable"):
        return []
    
    # Create instrumented version
    instrumented = ctx.actions.declare_file("instrumented_{}".format(
        target.label.name
    ))
    
    # ... create instrumentation action ...
    
    return [
        OutputGroupInfo(
            coverage = depset([instrumented]),
        ),
    ]

code_coverage_aspect = aspect(
    implementation = code_coverage_aspect_impl,
    attr_aspects = ["deps"],  # Apply to deps too
)
```

---

## Best Practices

### 1. Declare Dependencies Explicitly

```starlark
# ❌ Bad: tool is a string
ctx.actions.run(executable = "gcc", ...)

# ✅ Good: tool is a declared dependency
ctx.executable.compiler
```

### 2. Use Sandboxing

```starlark
# ❌ Bad: access system files
ctx.actions.run(
    command = "gcc -I/usr/include ...",
)

# ✅ Good: declare all inputs
ctx.actions.run(
    executable = ctx.executable.gcc,
    arguments = [
        "-I" + ctx.attr.libc_include,
        input_file.path,
    ],
    inputs = [input_file, libc_headers],
)
```

### 3. Make Outputs Deterministic

```starlark
# ❌ Bad: includes timestamp
ctx.actions.run(
    command = "echo 'Built at {}' > {}".format(
        datetime.now(), output.path
    ),
)

# ✅ Good: deterministic
ctx.actions.run(
    command = "echo 'Built' > {}".format(output.path),
)
```

### 4. Provide Good Error Messages

```starlark
def my_rule_impl(ctx):
    if not ctx.attr.tool:
        fail(
            "my_rule requires 'tool' attribute. "
            "Did you forget to add it to your BUILD file?"
        )
```

---

## Testing Custom Rules

```starlark
# test.bzl
def my_rule_test():
    """Test my custom rule."""
    
    # Create test targets
    my_custom_rule(
        name = "test_case_1",
        srcs = ["test_input.txt"],
    )
    
    # Use Bazel's testing infrastructure
    # (This is simplified; real tests are more complex)
```

---

## When to Write Custom Rules vs genrule

**Write custom rules when:**
- ✅ Reusable across many targets
- ✅ Complex logic (not a one-liner)
- ✅ Needs special integration (IDE support)
- ✅ Performance matters

**Use genrule when:**
- ✅ One-off build step
- ✅ Simple command
- ✅ Quick prototype

```starlark
# genrule: quick and dirty
genrule(
    name = "process",
    srcs = ["input.txt"],
    outs = ["output.txt"],
    cmd = "cat $(SRCS) | tr a-z A-Z > $(OUTS)",
)

# Custom rule: reusable and type-safe
upper_case_rule(
    name = "process",
    srcs = ["input.txt"],
)
```

---

## Resources for Going Deeper

To truly master Bazel extensions:

1. **Read the rulesets:** Study `rules_python`, `rules_cc`, `rules_go`
   - See how professionals structure rules
   - Learn patterns and best practices

2. **Understand the Starlark language:** It's Python-like but has constraints
   - No side effects (functional)
   - No infinite loops (Bazel would hang)
   - All data immutable (prevents bugs)

3. **Study real-world examples:**
   - Google's internal Bazel extensions
   - Community rulesets on GitHub

4. **Write rules incrementally:**
   - Start with `genrule` + wrapper script
   - Move to custom rule when you understand the pattern
   - Refine based on usage

---

## The Deeper Point

Writing custom Bazel rules requires understanding:
- **What makes Bazel work:** Determinism, isolation, parallelization
- **How to compose systems:** Providers, aspects, rule interactions
- **Systems thinking:** How your rule fits into the larger build graph

This is the "bottom layer engineering practice" that masters build systems. It's where the real power lies.

---

## See Also

- [[concepts/advanced/bazel-design-decisions]] — Why Bazel's constraints exist
- [[reference/bazel-vs-cmake]] — Why artifact-based systems are more extensible
- [Bazel Ruleset Development Guide](https://bazel.build/extending/rules) — Official docs
