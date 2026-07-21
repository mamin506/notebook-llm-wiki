---
title: "Repository Rules"
category: "concepts"
level: "advanced"
status: "growing"
sources: ["Repository Rules.md"]
tags: ["rules", "external-deps", "advanced"]
related: ["[[reference/external-dependencies]]", "[[concepts/fundamentals/labels]]", "[[concepts/advanced/macros]]"]
last_updated: "2026-07-19"
graph-group: "concepts"
---

# Repository Rules

Repository rules generate external repositories on demand. Unlike build rules (which generate artifacts), repository rules generate entire directories containing source code and BUILD files.

## Core Concept

A **repository rule** is a Starlark function that:
1. Takes attributes (like URL, SHA256, path, etc.)
2. Executes during the **Loading Phase** (before analysis)
3. Downloads/generates a repository directory
4. Returns a marker that the repo was successfully created

Example:
```python
# Define a repository rule
http_archive = repository_rule(
    implementation=_impl,
    attrs={
        "url": attr.string(mandatory=True),
        "sha256": attr.string(mandatory=True),
    }
)

# Use it in MODULE.bazel
http_archive(
    name = "zlib",
    url = "https://github.com/madler/zlib/archive/v1.2.11.tar.gz",
    sha256 = "c3e5e9f...",
)
```

## Key Differences from Build Rules

| Aspect | Build Rule | Repository Rule |
|--------|-----------|-----------------|
| **Input** | Targets from other rules | External source (URL, git, etc.) |
| **Output** | Artifacts (binaries, libraries) | Repository (directory with BUILD files) |
| **Execution Phase** | Analysis → Execution | Loading (before analysis) |
| **Hermetic** | Must be (sandboxed) | Often non-hermetic (network, filesystem) |
| **Caching** | Always re-run if input changes | Re-run only if attributes/env/watched files change |

## Anatomy of a Repository Rule

### Definition

```python
my_repo_rule = repository_rule(
    implementation=_impl,
    attrs={
        "url": attr.string(mandatory=True),
        "sha256": attr.string(mandatory=True),
        "strip_prefix": attr.string(default=""),
    },
    environ=["http_proxy", "https_proxy"],  # Watch these env vars
    doc="Fetch and extract an archive",
)
```

### Implementation Function

```python
def _impl(repository_ctx):
    # Access attributes
    url = repository_ctx.attr.url
    sha = repository_ctx.attr.sha256
    
    # Download and validate
    repository_ctx.download(
        url=url,
        output="archive.tar.gz",
        sha256=sha,
    )
    
    # Extract
    repository_ctx.extract(
        archive="archive.tar.gz",
        stripPrefix=repository_ctx.attr.strip_prefix,
    )
    
    # Create BUILD file(s)
    repository_ctx.file("BUILD", """
exports_files(["lib.a", "include/"])
""")
    
    # Return reproducibility info (optional)
    return None  # Means: "this rule is already reproducible"
```

## Important Execution Timing

**When is the implementation function executed?**

Repository rules execute **lazily** when their outputs are first needed:

```bash
# This doesn't fetch @zlib yet
bazel query //foo

# But this does (because //foo depends on @zlib)
bazel build //foo
```

**Re-fetching happens only if:**

1. Attributes change:
   ```python
   http_archive(name="zlib", url="...")  # Old URL
   # Change URL → repo refetches
   ```

2. Starlark code changes:
   ```python
   # Change the _impl function → repo refetches
   ```

3. Environment variables change (if declared with `environ`):
   ```python
   repository_rule(..., environ=["http_proxy"])
   # Change $http_proxy → repo refetches
   ```

4. Watched files/paths change (if using `repository_ctx.watch()`):
   ```python
   repository_ctx.watch("local_config.txt")
   # Modify local_config.txt → repo refetches
   ```

5. Force refetch:
   ```bash
   bazel fetch --force --all
   bazel fetch --force --configure  # Only repos with configure=True
   ```

## Common Repository APIs

### Download & Extract

```python
# Download from URL
repository_ctx.download(
    url="https://example.com/archive.tar.gz",
    output="archive.tar.gz",
    sha256="...",  # Verify integrity
    canonical_id="v1.0",  # For reproducibility tracking
)

# Extract archive
repository_ctx.extract(
    archive="archive.tar.gz",
    stripPrefix="folder-v1.0/",
)
```

### Execute Commands

```python
# Run shell command
result = repository_ctx.execute(
    ["git", "rev-parse", "HEAD"],
    working_directory=".",
)

# Access stdout, stderr, return_code
commit = result.stdout.strip()
```

### File Operations

```python
# Create file
repository_ctx.file("BUILD", """
cc_library(name="lib", ...)
""")

# Symlink
repository_ctx.symlink("/path/to/real", "symlink_path")

# Read file
content = repository_ctx.read("local_file.txt")
```

### Watching Paths

```python
# Re-fetch if this file changes
repository_ctx.watch("local_config.txt")

# Re-fetch if anything in directory changes
repository_ctx.watch_tree("configs/")
```

## Common Use Cases

### 1. Download Pre-Built Binary

```python
def _http_archive_impl(ctx):
    ctx.download_and_extract(
        url=ctx.attr.url,
        sha256=ctx.attr.sha256,
    )
    ctx.file("BUILD", """
exports_files(["bin", "lib"])
""")

http_archive = repository_rule(
    implementation=_http_archive_impl,
    attrs={
        "url": attr.string(mandatory=True),
        "sha256": attr.string(mandatory=True),
    }
)
```

### 2. Clone Git Repository

```python
def _git_repo_impl(ctx):
    ctx.download_and_extract(
        url=ctx.attr.url,
        type="zip",
        stripPrefix=ctx.attr.branch,
    )

git_repository = repository_rule(
    implementation=_git_repo_impl,
    attrs={
        "url": attr.string(mandatory=True),
        "branch": attr.string(default="main"),
    }
)
```

### 3. Auto-Configure Based on Host

```python
def _configure_impl(ctx):
    # Inspect local system
    result = ctx.execute(["gcc", "--version"])
    gcc_version = result.stdout
    
    # Generate BUILD file based on host
    ctx.file("BUILD", """
toolchain(
    name = "gcc",
    toolchain = ":gcc_config",
)
cc_toolchain(
    name = "gcc_config",
    compiler = "{}",
)
""".format(gcc_version))

cc_configure = repository_rule(
    implementation=_configure_impl,
    configure=True,  # Re-fetch on `bazel fetch --force --configure`
)
```

## Migration to Bazel Modules

**Modern approach:** Use Bazel modules (`MODULE.bazel`) and registries instead of raw `repository_rule`:

```python
# Old (raw repository_rule)
http_archive(name="zlib", url="...", sha256="...")

# Modern (Bazel modules)
bazel_dep(name="zlib", version="1.2.11")
```

Repository rules are still necessary for:
- Custom repository logic (local repos, git clones)
- Integrating legacy build systems
- Auto-configuration
- Private/corporate repositories

---

## Best Practices

1. **Always use SHA256 hashes** for downloaded artifacts
2. **Declare `environ` variables** if your rule depends on env vars
3. **Use `repository_ctx.watch()` carefully** — it can cause frequent re-fetches
4. **Provide meaningful error messages** when downloads fail
5. **Consider `configure=True`** if your rule depends on the host machine
6. **Prefer Bazel modules** for new projects (less boilerplate)
7. **Make repos reproducible** — return commit hashes instead of floating branches
