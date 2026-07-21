---
title: "Python in Bazel"
category: "languages"
level: "fundamentals"
status: "growing"
sources: ["Python Rules.md"]
tags: ["#python", "#rules", "#language-specific"]
related: ["[[concepts/fundamentals/targets]]", "[[reference/general-rules]]", "[[patterns/dependency-management]]"]
last_updated: "2026-07-19"
graph-group: "languages"
---

# Python in Bazel

Python support in Bazel uses the `rules_python` ruleset to enable Pythonic monorepos with hermetic, cacheable builds.

## Core Python Rules

### py_library

Defines a reusable Python library (set of `.py` modules and packages).

**Key Attributes:**
- `srcs`: Python source files (`.py`). Library targets go in `deps`, not `srcs`.
- `deps`: Other `py_library` targets this library depends on
- `data`: Runtime data files (not imports; think data files read by the code at runtime)
- `imports`: Directories to add to `PYTHONPATH` for this library and dependents (useful for packages)
- `precompile`: Controls whether `.py` files are compiled to `.pyc` at build time (`"inherit"`, `"enabled"`, `"disabled"`)

**Example:**
```python
py_library(
    name = "mylib",
    srcs = ["mylib.py"],
    deps = [":helper"],
)
```

### py_binary

Creates a runnable Python binary/script.

**Key Attributes:**
- `srcs`: Python source files
- `main`: The entry point `.py` file (defaults to `name.py` if not specified)
- `main_module`: Module name to execute (alternative to `main`; useful for running `-m module_name`)
- `deps`: Dependencies (usually `py_library` targets)
- `data`: Runtime data files
- `interpreter_args`: Arguments to pass to the Python interpreter itself

**Example:**
```python
py_binary(
    name = "myapp",
    srcs = ["myapp.py"],
    main = "myapp.py",
    deps = [":mylib"],
)
```

### py_test

Runs Python tests (pytest, unittest, or any test runner).

**Key Attributes:**
- `srcs`: Test files
- `deps`: Libraries under test (and test dependencies)
- `data`: Test data files
- Similar compilation/runtime options as `py_binary`

**Example:**
```python
py_test(
    name = "mylib_test",
    srcs = ["mylib_test.py"],
    deps = [":mylib"],
)
```

## Python Runtime

### py_runtime

Specifies which Python interpreter to use (in-build or platform-based).

**Types:**
- **Platform runtime**: References a system-installed interpreter via absolute path (`interpreter_path`)
- **In-build runtime**: References an executable target or files that act as the Python interpreter

**Example (platform):**
```python
py_runtime(
    name = "python3",
    interpreter_path = "/usr/bin/python3",
)
```

**Example (in-build):**
```python
py_runtime(
    name = "python3_hermetic",
    interpreter = ":python_binary",  # target that produces a Python binary
    files = glob(["python/**"]),      # supporting files
)
```

## Advanced Features

### Precompilation

By default, Python sources are interpreted at runtime. Bazel can optionally precompile `.py` to `.pyc`:

- `precompile = "enabled"`: Always compile `.py` → `.pyc` at build time
- `precompile = "inherit"`: Let downstream target decide (default)
- `precompile = "disabled"`: Never compile

Use `--precompile` flag or per-target `precompile` attribute. Precompiled binaries run slightly faster but increase build time.

### Type Checking

Declare type stubs separately from runtime code:

- `pyi_srcs`: Type definition files (`.pyi` files)
- `pyi_deps`: Packages providing type definitions (build-time only)

These are not included in the final binary.

### Virtual Environments (venv)

The `rules_python` ruleset can create isolated virtual environments at build time. The `imports` attribute determines the venv layout.

## Common Patterns

### Organizing Python Packages

Put each package in its own directory with a `BUILD` file:

```
myproject/
├── BUILD
├── __init__.py
├── subpkg1/
│   ├── BUILD
│   ├── __init__.py
│   └── module1.py
└── subpkg2/
    ├── BUILD
    ├── __init__.py
    └── module2.py
```

Each `BUILD` file declares targets for that package:

```python
py_library(
    name = "subpkg1",
    srcs = ["__init__.py", "module1.py"],
    visibility = ["//visibility:public"],
)
```

### Depending on External Packages

Use Bazel modules or `pip.parse()` to bring in third-party Python packages. Then depend on them:

```python
py_binary(
    name = "myapp",
    srcs = ["myapp.py"],
    deps = [
        "//myproject/subpkg1",
        "@pip//requests",  # from pip.parse()
    ],
)
```

### Testing with pytest

`py_test` rules work with pytest out of the box:

```python
py_test(
    name = "integration_test",
    srcs = ["test_integration.py"],
    deps = [
        ":mylib",
        "@pip//pytest",
    ],
    env = {
        "TEST_TIMEOUT": "60",
    },
)
```

Bazel will run pytest on the test file(s) and report results.

## Key Build Options

- `--python_version=3.11`: Specify Python version (must match a configured `py_runtime`)
- `--precompile`: Force precompilation of all Python targets
- `--test_output=all`: Show test stdout (useful for debugging tests)
- `--test_filter=pattern`: Run only matching tests

## Common Issues

**Issue: "No module named X"**
- Add missing import to `deps` (or `pyi_deps` for type-only imports)
- Ensure the module has an `__init__.py` if using implicit namespace packages

**Issue: "Relative import beyond top level"**
- Bazel's runfiles model doesn't always support relative imports cleanly
- Use absolute imports instead: `from mypackage import module`

**Issue: Data files not found at runtime**
- Add data files to `data` attribute, not `srcs`
- Access via `pkg_resources` or `importlib.resources` (Python 3.7+)

## See Also

- [[patterns/dependency-management]] — Managing Python dependencies
- [[reference/build-options]] — Build and test options
- [[concepts/fundamentals/targets]] — Understanding target concepts
