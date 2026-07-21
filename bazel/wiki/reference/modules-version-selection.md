---
title: "Bazel Modules: Version Selection & Overrides"
category: "reference"
level: "intermediate"
status: "growing"
sources: ["Bazel modules.md"]
tags: ["modules", "versioning", "dependencies", "#recommended"]
related: ["[[concepts/fundamentals/module.bazel]]", "[[reference/external-dependencies]]", "[[concepts/advanced/module-extensions]]"]
last_updated: "2026-07-19"
graph-group: "reference"
---

# Bazel Modules: Version Selection & Overrides

Modern Bazel dependency management using the MODULE.bazel system with version resolution and override strategies.

## Bazel Module Basics

A **Bazel module** is a versioned Bazel project. Every module has a `MODULE.bazel` file:

```python
# MODULE.bazel
module(name = "my-project", version = "1.0.5")

# Declare dependencies
bazel_dep(name = "rules_cc", version = "0.0.1")
bazel_dep(name = "protobuf", version = "3.19.0")
```

Key points:
- Module name + version uniquely identify a module
- `bazel_dep` declares direct dependencies (like `import` in other languages)
- Bazel loads the dependency's `MODULE.bazel` to discover transitive dependencies
- Only the root module's overrides affect resolution (overrides in dependencies are ignored)

---

## Version Formats

Bazel uses a **relaxed SemVer** that's more flexible than strict SemVer:

| Aspect | Strict SemVer | Bazel Modules |
|--------|---|---|
| Segments | Must be 3 (MAJOR.MINOR.PATCH) | Any number allowed |
| Character | Digits only | Digits + letters allowed |
| Semantics | Enforced | Not enforced |
| Examples | `1.0.0`, `2.3.4` | `1.0`, `20210324.2`, `1.2.3-alpha` |

Examples of valid Bazel module versions:
- `1.0` — Two-segment version
- `20210324.2` — Date-based version (like Abseil)
- `1.2.3-rc1` — Prerelease version
- `1.2.3+build.123` — Build metadata

**Comparison Rule:** Any valid SemVer version compares the same in Bazel modules as in SemVer.

---

## Version Selection: Minimal Version Selection (MVS)

When multiple versions of the same module are requested, Bazel uses the **Minimal Version Selection** (MVS) algorithm (borrowed from Go modules).

### The Diamond Dependency Problem

Consider this dependency graph:

```
MyApp 1.0
   /     \
LibA 1.0  LibB 1.1
  |        |
LibC 1.0  LibC 1.1
```

**Which version of LibC?** With MVS, Bazel picks **LibC 1.1** (the highest version specified by any dependent):

- LibA requests LibC 1.0
- LibB requests LibC 1.1
- MVS selects 1.1 (highest specified)

### Why MVS?

1. **Reproducible:** Same dependencies always resolve to same versions
2. **Minimal:** Picks the lowest version that satisfies all constraints
3. **Backwards compatible:** Assumes all versions are backwards compatible

### Yanked Versions

Registries can **yank** versions (mark them as unsafe/broken):

```bash
# Bazel rejects yanked versions
bazel build //foo
# Error: version 1.2.3 of module X is yanked

# Override to allow (use with caution!)
bazel build --allow_yanked_versions //foo
```

---

## Overrides

**Overrides** (only in root module) alter version resolution for dependencies:

```python
# MODULE.bazel (root module only)
module(name = "my-app", version = "1.0")

bazel_dep(name = "rules_cc", version = "0.0.1")
bazel_dep(name = "protobuf", version = "3.19.0")

# Overrides modify how dependencies resolve
single_version_override(name = "rules_cc", version = "0.0.2")
```

### Single-Version Override

Pin a dependency to a specific version:

```python
single_version_override(
    name = "protobuf",
    version = "3.20.0",  # Force this version instead of 3.19.0
)
```

Use cases:
- Security patch (upgrade to patched version)
- Incompatibility workaround (pin to known-good version)
- Performance fix (upgrade specific module)

#### Override from Specific Registry

```python
single_version_override(
    name = "custom_lib",
    version = "1.0",
    registry = "https://my-private-registry.com",
)
```

#### Apply Patches

```python
single_version_override(
    name = "third_party",
    version = "1.0",
    patches = [
        "//patches:fix_compilation.patch",
        "//patches:security_fix.patch",
    ],
    patch_strip = 1,  # Number of directories to strip in patches
)
```

### Multiple-Version Override

Allow multiple versions of the same module to coexist:

```python
multiple_version_override(
    name = "old_lib",
    versions = ["1.3", "1.7", "2.0"],
)
```

Example resolution:
- Dependency requests: 1.1, 1.3, 1.5, 1.7, 2.0
- Override allows: 1.3, 1.7, 2.0
- Result: 1.1 → 1.3, 1.5 → 1.7, others unchanged

Use case: Gradual migration when modules aren't fully compatible

### Non-Registry Overrides

Use local versions instead of registries:

#### Archive Override

```python
archive_override(
    name = "my_lib",
    urls = ["file:///local/my_lib-1.0.tar.gz"],
    strip_prefix = "my_lib-1.0",
    integrity = "sha256-...",
)
```

#### Git Override

```python
git_override(
    name = "rules_foo",
    remote = "https://github.com/example/rules_foo.git",
    commit = "abc123def456",
    patch_strip = 1,
    patches = ["//patches:fix.patch"],
)
```

#### Local Path Override

```python
local_path_override(
    name = "local_lib",
    path = "/path/to/local/lib",
)
```

Use case: Development (use local version while iterating)

---

## Repository Names & Strict Dependencies

### Apparent vs Canonical Names

**Apparent Name** (what you use in labels):
- Default: module name (e.g., `@protobuf`)
- Customizable: `bazel_dep(name="protobuf", repo_name="my_protobuf")`
- Only direct dependencies visible

**Canonical Name** (internal):
- Single version: `protobuf+3.19.0`
- Multiple versions: `old_lib+1.3`, `old_lib+1.7`, etc.
- Subject to change—don't hard-code!

### Strict Dependencies

You can only depend on **direct dependencies**, not transitive ones:

```python
# MODULE.bazel
bazel_dep(name = "rules_cc", version = "0.0.1")

# rules_cc depends on bazel_skylib, but:
# ❌ ERROR: @bazel_skylib is not a direct dependency
# ✅ OK: @rules_cc is a direct dependency
```

This prevents accidental breakages when transitive dependencies change.

### Getting Canonical Names

Don't hard-code canonical names. Instead:

```python
# In BUILD / .bzl files:
Label("@protobuf").repo_name

# When looking up runfiles:
$(rlocationpath //my/target)

# From external tools (IDE, language server):
bazel mod dump_repo_mapping --module=my-app
```

---

## Best Practices

1. **Use version constraints:** Specify meaningful versions in `bazel_dep`
2. **Document overrides:** Explain why each override exists (comment in MODULE.bazel)
3. **Avoid excessive patches:** If patching often, consider contributing upstream
4. **Test before yanking:** Ensure versions work before yanking
5. **Use local_path_override for dev:** Easy to iterate on local modules
6. **Pin critical dependencies:** For security-critical libraries, be explicit

---

## Troubleshooting

### "Module version X is yanked"
```bash
# Either upgrade:
bazel_dep(name = "foo", version = "2.0.0")

# Or explicitly allow (temporary):
bazel build --allow_yanked_versions
```

### "Ambiguous module resolution"
Use `single_version_override` or `multiple_version_override` to clarify.

### "Not a direct dependency"
Only depend on modules you directly declare in `bazel_dep`. Use `use_extension` and `use_repo` for indirect access.
