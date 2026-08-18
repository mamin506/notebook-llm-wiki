---
title: "Writing Custom Rules in Starlark"
category: "concepts"
level: "advanced"
status: "growing"
sources: ["Rules.md"]
tags: ["rules", "starlark", "advanced", "implementation", "providers", "actions"]
related: ["[[concepts/fundamentals/rules]]", "[[reference/general-rules]]", "[[experiments/extending-bazel]]"]
last_updated: "2026-07-30"
---

# Writing Custom Rules in Starlark

This page shows how to implement custom build rules using Starlark. For an introduction to rules, see [[concepts/fundamentals/rules]].

---

## Rule Creation

### Basic Rule Definition

Define a rule in a `.bzl` file using the `rule()` function:

```python
# In rules.bzl
example_library = rule(
    implementation = _example_library_impl,
    attrs = {
        "srcs": attr.label_list(allow_files = [".example"]),
        "hdrs": attr.label_list(allow_files = [".header"]),
        "deps": attr.label_list(providers = [ExampleInfo]),
        "data": attr.label_list(allow_files = True),
    },
)
```

Then use in `BUILD` files:

```python
load("//tools:rules.bzl", "example_library")

example_library(
    name = "my_target",
    srcs = ["input.example"],
    deps = [":other_target"],
)
```

---

## Attributes

Attributes are the rule's parameters. They define what values a user can provide when instantiating the rule.

### Dependency Attributes

Attributes that refer to other targets:

```python
attrs = {
    "srcs": attr.label_list(allow_files = [".example"]),  # Source files
    "hdrs": attr.label_list(allow_files = [".header"]),   # Headers
    "deps": attr.label_list(providers = [ExampleInfo]),   # Dependencies
    "data": attr.label_list(allow_files = True),          # Runtime data
}
```

These are represented as lists of `Target` objects in the implementation function.

### Output Attributes

Declare output files that the rule produces:

```python
attrs = {
    "out": attr.output(),           # Single output file
    "outs": attr.output_list(),     # Multiple output files
}
```

Output attributes allow users to name the output:

```python
example_library(
    name = "mytarget",
    out = "custom_output.o",        # User specifies output name
)
```

### Private Attributes (Implicit Dependencies)

Attributes prefixed with `_` are private and provide implicit dependencies:

```python
attrs = {
    "_compiler": attr.label(
        default = Label("//tools:example_compiler"),
        allow_single_file = True,
        executable = True,
        cfg = "exec",
    ),
}
```

Private attributes:
- Must have default values
- Cannot be overridden by users
- Commonly used for tool dependencies (compiler, linker, etc.)

---

## Implementation Function

The implementation function is called during the **analysis phase** and must:
1. Read inputs and attributes
2. Gather dependencies via providers
3. Register actions
4. Return providers

### Function Signature

```python
def _my_rule_impl(ctx):
    # ctx: RuleContext object
    # Returns: list of providers
    return [DefaultInfo(...), MyInfo(...)]
```

### Accessing Attributes

```python
def _my_rule_impl(ctx):
    # Source files (from "srcs" attribute)
    srcs = ctx.files.srcs                    # List of File objects
    
    # Dependencies (from "deps" attribute)
    deps = ctx.attr.deps                     # List of Target objects
    
    # Get providers from dependencies
    for dep in ctx.attr.deps:
        info = dep[ExampleInfo]              # Get ExampleInfo provider
```

### Working with Files

Files are represented by `File` objects (source or generated):

```python
def _my_rule_impl(ctx):
    srcs = ctx.files.srcs               # List of source files
    
    # Declare a new output file
    output = ctx.actions.declare_file(ctx.label.name + ".output")
    
    # Declare a directory output
    output_dir = ctx.actions.declare_directory("output_dir")
    
    # Use ctx.outputs for predeclared outputs
    out_file = ctx.outputs.out          # From output attribute
```

---

## Actions

Actions describe how to generate outputs from inputs. Register actions in the implementation function:

### Running an Executable

```python
ctx.actions.run(
    executable = ctx.executable._compiler,
    arguments = [arg1, arg2, ...],
    inputs = depset([...input files...]),
    outputs = [output_file],
    mnemonic = "MyCompile",             # Action label (for logging)
)
```

### Running a Shell Command

```python
ctx.actions.run_shell(
    command = "cat $1 > $2",
    arguments = [input_file.path, output_file.path],
    inputs = [input_file],
    outputs = [output_file],
)
```

### Writing a File

```python
ctx.actions.write(
    output = config_file,
    content = "CONFIG=value\n",
)
```

### Building Arguments Efficiently

Use `ctx.actions.args()` to avoid flattening depsets:

```python
args = ctx.actions.args()
args.add_joined("-h", headers, join_with = ",")
args.add_joined("-s", srcs, join_with = ",")
args.add("-o", output_file)

ctx.actions.run(
    arguments = [args],
    ...
)
```

### Action Requirements

Actions must:
- **List all inputs** (used by action)
- **Declare all outputs** (created by action)
- **Be deterministic** (same inputs → same outputs)
- **Not access environment** (username, clock, network, etc.)

```python
def _my_rule_impl(ctx):
    ...
    transitive_headers = [dep[ExampleInfo].headers for dep in ctx.attr.deps]
    headers = depset(ctx.files.hdrs, transitive = transitive_headers)
    srcs = ctx.files.srcs
    inputs = depset(srcs, transitive = [headers])      # All inputs
    output_file = ctx.actions.declare_file(...)
    
    ctx.actions.run(
        inputs = inputs,                               # Must list all inputs
        outputs = [output_file],                       # Must list all outputs
        ...
    )
```

---

## Providers

Providers are pieces of information that a rule exposes to consumers. They enable communication between rules.

### DefaultInfo

Every rule should provide `DefaultInfo`:

```python
def _my_rule_impl(ctx):
    output = ctx.actions.declare_file(...)
    ...
    return [
        DefaultInfo(
            files = depset([output]),                  # Default outputs
            runfiles = ctx.runfiles(files=[...]),      # Runtime files
        )
    ]
```

### Custom Providers

Define custom providers to expose rule-specific information:

```python
# Define a provider
ExampleInfo = provider(
    "Information about Example compilation",
    fields = {
        "headers": "depset of header Files",
        "files_to_link": "depset of object Files",
    },
)

# Use in implementation
def _my_rule_impl(ctx):
    ...
    return [
        DefaultInfo(...),
        ExampleInfo(
            headers = depset(ctx.files.hdrs),
            files_to_link = depset([output_file]),
        )
    ]
```

### Accessing Providers from Dependencies

```python
def _my_rule_impl(ctx):
    # Collect headers from all dependencies
    transitive_headers = [dep[ExampleInfo].headers for dep in ctx.attr.deps]
    all_headers = depset(
        ctx.files.hdrs,
        transitive = transitive_headers
    )
```

### Runfiles

Runfiles are files needed at runtime (not build time):

```python
def _my_rule_impl(ctx):
    # Create runfiles from data attribute
    runfiles = ctx.runfiles(files = ctx.files.data)
    
    # Merge in runfiles from dependencies
    transitive_runfiles = []
    for target in ctx.attr.deps:
        transitive_runfiles.append(target[DefaultInfo].default_runfiles)
    runfiles = runfiles.merge_all(transitive_runfiles)
    
    return [
        DefaultInfo(..., runfiles = runfiles),
        ...
    ]
```

---

## Executable and Test Rules

### Executable Rules

Rules that produce executables must set `executable = True`:

```python
my_binary_rule = rule(
    implementation = _my_binary_impl,
    executable = True,
    attrs = {...},
)

def _my_binary_impl(ctx):
    executable_file = ctx.actions.declare_file(...)
    ...
    return [
        DefaultInfo(executable = executable_file, ...)
    ]
```

### Test Rules

Test rules must set `test = True` and have names ending in `_test`:

```python
my_test_rule = rule(
    implementation = _my_test_impl,
    test = True,
    attrs = {...},
)
```

Test rules automatically get test-specific attributes and providers.

---

## Common Patterns

### Collecting Transitive Dependencies

```python
def _my_rule_impl(ctx):
    # Collect transitive information using depsets
    transitive_headers = [dep[MyInfo].headers for dep in ctx.attr.deps]
    all_headers = depset(
        ctx.files.hdrs,
        transitive = transitive_headers
    )
```

### Accessing Compilation Context

```python
def _my_rule_impl(ctx):
    # From CC dependencies
    for dep in ctx.attr.deps:
        cc_info = dep[CcInfo]
        # cc_info contains includes, libs, etc.
```

### Implicit Tool Dependencies

```python
attrs = {
    "_compiler": attr.label(
        default = Label("//tools:compiler"),
        executable = True,
        cfg = "exec",
    ),
}

def _my_rule_impl(ctx):
    compiler = ctx.executable._compiler
    ctx.actions.run(executable = compiler, ...)
```

---

## Execution Requirements

Control how actions execute using execution_requirements:

```python
ctx.actions.run_shell(
    command = "...",
    execution_requirements = {
        "local": "1",              # Must run locally
        "no-remote": "1",          # No remote caching
        "requires-fakeroot": "1",  # Needs root
    },
    ...
)
```

See [[reference/execution-tags-and-caching]] for complete list.

---

## Best Practices

1. **Use depsets** for efficiency when collecting transitive information
2. **Provide default outputs** even if not directly used (helps with debugging)
3. **Make attributes private** (prefix with `_`) when users shouldn't override them
4. **Document attributes** with clear descriptions
5. **Use meaningful mnemonics** for actions (aid debugging)
6. **Avoid reading files** during analysis phase (only during execution)
7. **Consider platform/toolchain implications** when accessing tools

---

## See Also

- [[concepts/fundamentals/rules]] — Introduction to rules
- [[reference/execution-tags-and-caching]] — Controlling action execution
- [[experiments/extending-bazel]] — Deep dive into Bazel extensions
- [Bazel Rules Tutorial](https://bazel.build/rules/rules-tutorial) — Official interactive guide
