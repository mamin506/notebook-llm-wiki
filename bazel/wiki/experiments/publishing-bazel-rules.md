---
title: "Publishing Bazel Rules: From Private to Public"
category: "experiments"
level: "advanced"
status: "seedling"
sources: ["Deploying Rules.md"]
tags: ["publishing", "rulesets", "distribution", "github", "open-source", "advanced"]
related: ["[[concepts/advanced/writing-custom-rules]]", "[[experiments/extending-bazel]]", "[[concepts/fundamentals/module.bazel]]"]
last_updated: "2026-07-30"
---

# Publishing Bazel Rules: From Private to Public

For rule developers ready to share their work with the community.

---

## Why Publish Rules?

**Before publishing:**
```
You wrote a custom rule
└─ Works for your project
```

**After publishing:**
```
You published a ruleset
├─ Other projects discover and use it
├─ Community contributes improvements
├─ Your rule becomes an ecosystem standard
└─ Less duplication across projects
```

---

## Naming and Hosting

### Repository Naming Convention

**The standard:** `$ORGANIZATION/rules_$NAME`

**Examples:**
- `bazelbuild/rules_go` — Go support
- `bazelbuild/rules_python` — Python support
- `build_stack/rules_proto` — Protocol buffers (non-bazelbuild org)

**Why consistency matters:** Users expect `rules_something`, making discovery and understanding easier.

### Module Name

In `MODULE.bazel`, use:
- **For bazelbuild org:** `rules_<lang>` (e.g., `rules_mockascript`)
- **For other orgs:** `<org>_rules_<lang>` (e.g., `build_stack_rules_proto`)

```starlark
# bazelbuild org
module(name = "rules_python")

# Other org
module(name = "build_stack_rules_proto")
```

### GitHub Metadata

Make your ruleset discoverable:

```
Repository name:        bazelbuild/rules_go
Repository description: Go rules for Bazel
Tags:                   golang, bazel
README.md header:       Go rules for Bazel (link to https://bazel.build)
```

**Why:** Users searching "Bazel Go rules" will find you.

---

## Repository Structure

Standard layout for `rules_mockascript`:

```
rules_mockascript/
├── MODULE.bazel                # Module definition
├── README.md                   # Clear overview and API
├── LICENSE                     # Licensing (MIT, Apache 2.0, etc.)
├── mockascript/               # Language directory
│   ├── BUILD
│   ├── defs.bzl              # Rule definitions (public API)
│   ├── internal/
│   │   ├── BUILD
│   │   └── impl.bzl          # Implementation (private)
│   └── constraints/          # Custom platform constraints
│       └── BUILD
├── tests/
│   ├── BUILD
│   ├── integration_test.sh
│   └── unit_test.py
├── examples/                 # Optional: show common usage
│   ├── BUILD
│   ├── simple.mocs
│   ├── complex.mocs
│   └── test.mocs
├── docs/                     # Generated documentation
│   ├── README.md
│   └── rules.md
└── .github/
    └── workflows/
        ├── ci.yaml           # Run tests on PR
        └── release.yaml      # Publish on tag
```

### Key Directories Explained

**`mockascript/defs.bzl`** — Your public API:
```starlark
"""Public API for mockascript rules."""

load(":internal/impl.bzl", _mockascript_impl = "impl")

mockascript_binary = rule(
    implementation = _mockascript_impl,
    # ... public attrs ...
)

# Export what users see
```

**`mockascript/constraints/BUILD`** — Custom platform constraints:
```starlark
constraint_setting(name = "compiler")

constraint_value(
    name = "mockc",
    constraint_setting = ":compiler",
)
```

Use only if you need custom platform logic. Consider contributing to [bazelbuild/platforms](https://github.com/bazelbuild/platforms) for language-independent constraints.

**`tests/`** — Comprehensive test coverage:
```
tests/
├── basic_binary_test.sh      # Does mockascript_binary work?
├── library_test.sh            # Does mockascript_library work?
└── toolchain_test.sh          # Does toolchain resolution work?
```

**`examples/`** — Show common usage patterns:
```
examples/
├── hello_world/
│   ├── BUILD
│   └── hello.mocs
├── library_usage/
│   ├── BUILD
│   ├── lib.mocs
│   └── main.mocs
└── with_data/
    ├── BUILD
    ├── app.mocs
    └── data.txt
```

---

## Module Dependencies and Toolchain Registration

### Declaring Dependencies

In `MODULE.bazel`:

```starlark
module(name = "rules_mockascript", version = "1.0.0")

# Runtime dependencies
bazel_dep(name = "bazel_skylib", version = "1.4.0")
bazel_dep(name = "rules_cc", version = "0.1.1")  # If C-based tools

# Development dependencies
bazel_dep(name = "googletest", version = "1.14.0")
```

### Registering Toolchains

If your rules provide toolchains:

```starlark
# MODULE.bazel
module(name = "rules_mockascript")

# ... other deps ...

# Register toolchains
register_toolchains(
    "//mockascript/toolchain:all",
)
```

**Important:** Bazel analyzes all registered `toolchain` targets at analysis time. For complex repository rules:
- Keep `toolchain` registration lightweight (just registers targets)
- Move complex setup to separate repository (`mockascript_toolchain_impl`)
- Users fetch toolchain repository only when needed

**Example:** `rules_python` has:
- `rules_python` — Registers toolchains (lightweight)
- `python_source` — Actual Python interpreter setup (heavy, fetched only when needed)

---

## Documentation

### README.md

Your README is the first thing users see. Include:

```markdown
# Go rules for Bazel

Bazel rules for the [Go](https://golang.org/) programming language.

## Installation

Add to your `MODULE.bazel`:

```
bazel_dep(name = "rules_go", version = "0.42.0")
```

## Quick start

```starlark
load("@rules_go//go:def.bzl", "go_binary", "go_library")

go_library(
    name = "mylib",
    srcs = ["lib.go"],
)

go_binary(
    name = "myapp",
    srcs = ["main.go"],
    deps = [":mylib"],
)
```

## Rules

- [go_binary](docs/go_binary.md) — Build Go executable
- [go_library](docs/go_library.md) — Build Go library
- [go_test](docs/go_test.md) — Run Go tests

## Contributing

...
```

### Auto-Generated Documentation

Use [Stardoc](https://github.com/bazelbuild/stardoc) to auto-generate API docs from comments:

```starlark
# mockascript/defs.bzl
"""Rules for the Mockascript language."""

mockascript_binary = rule(
    doc = """
    Compile a mockascript program into an executable.

    Args:
        srcs: Source files to compile
        deps: Dependencies to link
        data: Runtime data files
    """,
    implementation = ...,
)
```

Stardoc generates `docs/rules.md` automatically.

**See:** [rules-template/docs](https://github.com/bazel-contrib/rules-template/tree/main/docs) for setup.

---

## CI/CD Pipeline

### GitHub Actions Setup

Use [bazel-contrib reusable workflows](https://github.com/bazel-contrib/bazel-examples/tree/main/.github/workflows):

**`.github/workflows/ci.yaml`** — Test on every PR and commit:
```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: bazel-contrib/setup-bazel@v0
      - run: bazel test //...
```

**`.github/workflows/release.yaml`** — Publish on tag:
```yaml
name: Release

on:
  push:
    tags:
      - v*

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: bazel-contrib/setup-bazel@v0
      - name: Build release
        run: bazel build //...
      - name: Create GitHub release
        run: gh release create ${{ github.ref }}
```

### bazel-ci Integration

If your rules are in the [bazelbuild](https://github.com/bazelbuild) organization, request to add them to [ci.bazel.build](http://ci.bazel.build/):

1. Open issue in [bazelbuild/continuous-integration](https://github.com/bazelbuild/continuous-integration/issues)
2. Template: "Request to add new project [PROJECT_NAME]"
3. Bazel CI team will integrate your tests into the central build farm

---

## Release Process

### Pre-Release Checklist

Before releasing version 1.0.0:

- [ ] Tests pass locally (`bazel test //...`)
- [ ] Tests pass in CI (GitHub Actions)
- [ ] Documentation is complete
- [ ] Examples all work
- [ ] CHANGELOG.md updated with new features
- [ ] README.md reflects latest features

### Creating a Release

1. **Update version in MODULE.bazel:**
   ```starlark
   module(name = "rules_mockascript", version = "1.0.0")
   ```

2. **Update CHANGELOG.md:**
   ```markdown
   ## 1.0.0 (2026-07-20)

   ### Features
   - Initial release with mockascript_binary and mockascript_library
   - Support for constraints
   - Documentation and examples

   ### Breaking Changes
   - None
   ```

3. **Tag and push:**
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```

4. **GitHub creates release automatically** (via release.yaml workflow)

### Release Announcement Snippet

Include in release notes for users to copy-paste:

```starlark
# In your MODULE.bazel
bazel_dep(name = "rules_mockascript", version = "1.0.0")
```

---

## Community Standards

### Why Not in Main Bazel Repo?

**Question:** Why are rules separate from Bazel itself?

**Answer:**
- ✅ **Clear ownership** — Each ruleset has its own maintainers
- ✅ **Independent versioning** — Rules can update without Bazel releases
- ✅ **Lower barrier to contribution** — Easier to get commit access to ruleset
- ✅ **User flexibility** — Users can use, replace, or downgrade rules independently
- ❌ **Trade-off** — Users must add dependency to `MODULE.bazel`

**History:** Bazel used to include rules in `/tools/build_rules`. That approach became unwieldy at scale. Now rules live in separate repositories.

### Contributing to platforms

If your constraints are language-independent, consider contributing to [bazelbuild/platforms](https://github.com/bazelbuild/platforms):

```starlark
# Instead of:
constraint_value(
    name = "gnu_compiler",
    constraint_setting = "@my_rules//constraints:compiler",
)

# Contribute to:
constraint_value(
    name = "gnu_compiler",
    constraint_setting = "@platforms//constraints:compiler",
)
```

**Benefit:** Ecosystem speaks one language; reduces duplication.

---

## Template Repository

**Don't start from scratch.** Use [bazel-contrib/rules-template](https://github.com/bazel-contrib/rules-template):

```bash
git clone https://github.com/bazel-contrib/rules-template.git rules_mylangu
cd rules_mylang

# The template includes:
# ✅ Correct directory structure
# ✅ MODULE.bazel setup
# ✅ CI/CD workflows (ci.yaml, release.yaml)
# ✅ Stardoc documentation setup
# ✅ Example tests
# ✅ LICENSE
```

Then customize for your language/tool.

---

## Example: Anatomy of rules_python

`rules_python` is a mature, well-organized ruleset. Reference it for:

**Structure:**
```
rules_python/
├── python/                    # Main language directory
│   ├── BUILD
│   ├── defs.bzl              # Public API
│   ├── repositories.bzl       # Repository rules
│   └── toolchains/           # Toolchain implementation
├── tests/
├── examples/
└── docs/
```

**Key files:**
- `MODULE.bazel` — Module definition, dependency registration
- `python/defs.bzl` — Public rules (`py_binary`, `py_library`, `py_test`)
- `python/toolchain/` — How Python is found/registered
- Documentation generated by Stardoc

**Release rhythm:** New version every 1-2 months, following Bazel's pace.

---

## Checklist: Before Publishing

- [ ] Repository created at `$ORG/rules_$NAME`
- [ ] Module name set in MODULE.bazel
- [ ] Directory structure follows convention
- [ ] README.md is clear and has quick-start example
- [ ] defs.bzl exports all public rules
- [ ] Tests exist and pass locally
- [ ] Tests pass in CI (GitHub Actions)
- [ ] Examples demonstrate common usage
- [ ] Documentation generated (Stardoc or manual)
- [ ] LICENSE file present
- [ ] CHANGELOG.md started
- [ ] Release snippet prepared for first tag

---

## See Also

- [[experiments/extending-bazel]] — How to write the rules before publishing them
- [[concepts/fundamentals/module.bazel]] — MODULE.bazel format
- [bazel-contrib/rules-template](https://github.com/bazel-contrib/rules-template) — Official template
- [Stardoc](https://github.com/bazelbuild/stardoc) — Auto-generate documentation
