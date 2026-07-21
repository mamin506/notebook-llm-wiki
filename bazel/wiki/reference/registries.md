---
title: "Bazel Registries and Module Discovery"
category: "reference"
level: "intermediate"
status: "seedling"
sources: ["Bazel registries.md"]
tags: ["#registries", "#modules", "#external-dependencies", "#bazel-central-registry"]
related: ["[[concepts/fundamentals/module.bazel]]", "[[reference/modules-version-selection]]", "[[reference/external-dependencies]]"]
last_updated: "2026-07-19"
graph-group: "reference"
---

# Bazel Registries and Module Discovery

Registries are databases of Bazel modules — they tell Bazel how to find and fetch external dependencies.

## Registry Overview

A **registry** is a local directory or static HTTP server containing:
- Module metadata (homepage, maintainers, versions)
- `MODULE.bazel` files for each version
- Instructions on how to fetch the source for each version

Bazel discovers dependencies by requesting module info from registries.

### Registry Format

All registries follow the **index registry** format:

```
registry-root/
├── bazel_registry.json        # Optional registry metadata
└── modules/
    ├── mymodule/
    │   ├── metadata.json      # Module versions, yanked versions
    │   ├── 1.0.0/
    │   │   ├── MODULE.bazel   # Module file for v1.0.0
    │   │   ├── source.json    # How to fetch v1.0.0 source
    │   │   └── patches/       # Optional: patches to apply
    │   └── 2.0.0/
    │       ├── MODULE.bazel
    │       ├── source.json
    │       └── overlay/       # Optional: overlay files
    └── another_module/
        ├── metadata.json
        └── 1.0.0/
            ├── MODULE.bazel
            └── source.json
```

## Registry Files

### bazel_registry.json

Optional metadata for the entire registry:

```json
{
  "mirrors": [
    "https://mirror1.example.com/",
    "https://mirror2.example.com/"
  ],
  "module_base_path": "modules"  // For local_path source type
}
```

**Fields:**
- `mirrors` — Mirror URLs for source archives (tried in order before original URL)
- `module_base_path` — Base path for modules with `local_path` type in source.json

### metadata.json (per-module)

Declares available versions and yanked versions:

```json
{
  "versions": ["1.0.0", "1.1.0", "2.0.0"],
  "yanked_versions": {
    "1.0.0": "Security vulnerability, use 1.1.0 or later",
    "2.0.0-beta": "Beta release, not for production"
  }
}
```

**Fields:**
- `versions` — List of available versions (should match directory names)
- `yanked_versions` — Versions to reject (with reason); Bazel will not resolve to yanked versions unless explicitly overridden

### MODULE.bazel

The `MODULE.bazel` file for each version. This is the **registry version**, not necessarily the one from the source archive (they can differ).

**Example:**
```python
module(
    name = "mylib",
    version = "1.0.0",
    compatibility_level = 1,
)

bazel_dep(name = "other_lib", version = "2.0")
```

### source.json

Describes how to fetch the source for a module version. Schema depends on `type`:

#### type: "archive" (default)

Fetch from an HTTP URL and extract:

```json
{
  "type": "archive",
  "url": "https://github.com/myorg/mylib/releases/download/v1.0.0/mylib-1.0.0.tar.gz",
  "integrity": "sha256-abc123...",
  "strip_prefix": "mylib-1.0.0",
  "patches": {
    "fix.patch": "sha256-def456..."
  },
  "patch_strip": 1
}
```

**Fields:**
- `url` — Archive URL
- `mirror_urls` — Fallback mirrors (optional)
- `integrity` — Subresource Integrity checksum
- `strip_prefix` — Directory prefix to remove after extraction
- `overlay` — Files to layer on top after extraction
- `patches` — Patch files to apply
- `patch_strip` — Strip N levels from patch paths (like `patch -p N`)
- `archive_type` — Archive format if not inferred from URL (e.g., `zip`, `tar.gz`)

#### type: "git_repository"

Clone from a Git repository:

```json
{
  "type": "git_repository",
  "remote": "https://github.com/myorg/mylib.git",
  "commit": "abc123def456...",
  "shallow_since": "2026-01-15",
  "patches": {
    "feature.patch": "sha256-..."
  }
}
```

**Fields:**
- `remote` — Git URL
- `commit` — Exact commit hash
- `tag` — Tag (alternative to commit)
- `shallow_since` — Shallow clone date (optional, for speed)
- `init_submodules` — Clone submodules
- `patches` — Patch files to apply

#### type: "local_path"

Reference a local directory:

```json
{
  "type": "local_path",
  "path": "../sibling_repo"
}
```

Used for monorepos or local development. Path is resolved relative to `module_base_path`.

## Bazel Central Registry (BCR)

The **Bazel Central Registry** is the official, community-maintained registry of Bazel modules.

**URL:** https://bcr.bazel.build/

**Browser:** https://registry.bazel.build/

**GitHub:** https://github.com/bazelbuild/bazel-central-registry

### BCR Structure

The BCR is a GitHub repository that follows the index registry format. It includes:

- Modules from the Bazel ecosystem
- Community-vetted, verified modules
- Strict quality requirements (maintainability, documentation, testing)

### Contributing to BCR

The community maintains the BCR. To contribute:

1. Fork `bazelbuild/bazel-central-registry`
2. Add/update module in `modules/<name>/<version>/`
3. Submit pull request with:
   - `MODULE.bazel` file
   - `source.json` file
   - `presubmit.yml` (test targets for CI validation)
4. BCR CI verifies the module works with other modules

See [BCR contribution guide](https://github.com/bazelbuild/bazel-central-registry/blob/main/docs/README.md).

## Configuring Registries

### Specifying Registries

Use the `--registry` flag to specify registry URL(s):

```bash
bazel build //mylib --registry=https://bcr.bazel.build --registry=https://my-company-registry.example.com
```

**.bazelrc:**
```
# Use company registry first, then BCR as fallback
common --registry=https://my-company-registry.example.com
common --registry=https://bcr.bazel.build
```

**Registry Priority:**
- Registries are searched in order
- First registry to have the module wins
- To disable BCR, don't add it to `--registry` flags

### Custom Registries

Host a private registry for internal modules:

```
my-registry/
├── bazel_registry.json
└── modules/
    └── internal-lib/
        ├── metadata.json
        └── 1.0.0/
            ├── MODULE.bazel
            └── source.json
```

In `.bazelrc`:
```
common --registry=file:///path/to/my-registry
common --registry=https://my-company.example.com/registry
common --registry=https://bcr.bazel.build
```

### GitHub-Hosted Registries

For registries on GitHub, use the **raw** URL:

```
# Correct (raw content)
common --registry=https://raw.githubusercontent.com/my-org/bazel-registry/main/

# Wrong (HTML view, doesn't work)
# --registry=https://github.com/my-org/bazel-registry/
```

## Modern Workflow

### Old Way (WORKSPACE + http_archive)

```python
# WORKSPACE
load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

http_archive(
    name = "openssl",
    url = "https://github.com/openssl/openssl/releases/download/openssl-1.1.1.../openssl-1.1.1....tar.gz",
    sha256 = "...",
)
```

**Problems:**
- Manual version management
- Version conflicts (no MVS)
- No transitive dependency resolution

### New Way (MODULE.bazel + Registries)

```python
# MODULE.bazel
bazel_dep(name = "openssl", version = "1.1.1")
bazel_dep(name = "zlib", version = "1.2.11")
```

**Benefits:**
- Declarative, clean
- Automatic version resolution (MVS)
- Transitive dependencies
- Yanked version protection
- Easier updates

## Troubleshooting

**Issue: "Module not found in any registry"**
- Check module name spelling
- Verify registry is accessible
- Check `--registry` flags are correct

**Issue: "Version not available in registry"**
- Version may not be in the registry yet
- Try a different version
- Check if version is yanked (use `--noenable_bzlmod` to bypass module system if urgent)

**Issue: "Cannot connect to registry"**
- Network connectivity issue
- Verify registry URL is correct
- Try with proxy/authentication if needed

## See Also

- [[concepts/fundamentals/module.bazel]] — MODULE.bazel file format
- [[reference/modules-version-selection]] — Version selection algorithm
- [[reference/external-dependencies]] — External dependency management
