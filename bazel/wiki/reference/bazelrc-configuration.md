---
title: ".bazelrc Configuration Files"
category: "reference"
level: "intermediate"
status: "growing"
sources: ["Write bazelrc configuration files.md"]
tags: ["configuration", "bazelrc", "workflow"]
related: ["[[reference/cli-reference]]", "[[patterns/build-file-style]]"]
last_updated: "2026-07-19"
graph-group: "reference"
---

# .bazelrc Configuration Files

Configure Bazel with persistent options across builds using `.bazelrc` files.

## Overview

`.bazelrc` files store command-line options that apply across builds, avoiding repetition of frequently-used flags.

**Without .bazelrc:**
```bash
bazel build --compilation_mode=opt --cxxopt="-std=c++17" --keep_going //foo
bazel test --compilation_mode=opt --cxxopt="-std=c++17" --keep_going //foo:tests
```

**With .bazelrc:**
```bash
# .bazelrc
build --compilation_mode=opt --cxxopt="-std=c++17" --keep_going

# Now just:
bazel build //foo
bazel test //foo:tests
```

---

## RC File Search Path

Bazel loads `.bazelrc` files in order (options in later files override earlier ones):

1. **System RC** — `/etc/bazel.bazelrc` (Linux/macOS) or `%ProgramData%\bazel.bazelrc` (Windows)
   - Set via environment or custom Bazel binary

2. **Workspace RC** — `.bazelrc` in workspace root (next to `MODULE.bazel`)
   - Checked into version control

3. **Home RC** — `$HOME/.bazelrc` (Linux/macOS) or `%USERPROFILE%\.bazelrc` (Windows)
   - Per-user configuration

4. **Environment RC** — Path(s) from `BAZELRC` environment variable
   - Comma-separated if multiple paths

5. **User-Specified RC** — `--bazelrc=file` on command line (can use multiple times)
   - Takes highest precedence

### Example Load Order

```bash
# Options loaded in this order (later overrides earlier):
# 1. /etc/bazel.bazelrc (system)
# 2. .bazelrc (workspace)
# 3. ~/.bazelrc (home)
# 4. $BAZELRC env var
# 5. --bazelrc=custom.rc
# 6. --bazelrc=override.rc

bazel build --bazelrc=custom.rc --bazelrc=override.rc //foo
```

### Disabling RC Files

Pass `/dev/null` to disable further RC loading:

```bash
bazel build --bazelrc=x.rc --bazelrc=/dev/null --bazelrc=z.rc //foo
# Only x.rc is loaded; z.rc is ignored
```

Use case: CI/release builds (avoid local .bazelrc pollution)

---

## Syntax

`.bazelrc` is a line-based text file:

```bash
# Comments start with #
# Empty lines are ignored

# Option format:
[command] [flag1] [flag2] ...

# Examples:
build --compilation_mode=opt
test --test_size_filters=small,medium
run --ui_event_filters=-info

# Multi-word values need quotes:
build --cxxopt="-std=c++17"
```

### Tokenization

Options are tokenized like Bourne shell:

```bash
# These are equivalent:
build --cxxopt="-std=c++17" --linkopt="-lm"
build --cxxopt "-std=c++17" --linkopt "-lm"

# Quotes needed if space in value:
common --action_env=MY_VAR="value with spaces"
```

---

## Command-Specific Sections

Options apply to specific commands:

```bash
# All commands
common --color=yes

# Only build command
build --compilation_mode=opt --cxxopt="-std=c++17"

# Only test command
test --test_output=streamed

# Only run command
run --ui_event_filters=-info

# Build and test (no separate run options)
# Use 'common' to apply to all
```

### Command Variants

```bash
# build:myconfig applies when --config=myconfig is passed
build:myconfig --compilation_mode=opt --cxxopt="-O3"

# test:debug applies to test command with --config=debug
test:debug --test_output=all --verbose_failures

# Usage:
bazel build --config=myconfig //foo
bazel test --config=debug //foo:tests
```

---

## Imports

Include other `.bazelrc` files:

```bash
# Require file to exist (fail if missing)
import %workspace%/configs/dev.rc

# Optional import (continue if missing)
try-import ~/.bazelrc.local

# Conditional import based on Bazel version
try-import-if-bazel-version >=6.0.0 %workspace%/configs/v6_features.rc
try-import-if-bazel-version <7.0.0 %workspace%/configs/legacy_flags.rc
```

### Version Operators

```bash
# Strictly greater than
try-import-if-bazel-version >6.0.0 configs/post_v6.rc

# Greater than or equal (common for features)
try-import-if-bazel-version >=6.0.0 configs/v6_features.rc

# Strictly less than
try-import-if-bazel-version <7.0.0 configs/legacy.rc

# Less than or equal
try-import-if-bazel-version <=5.4.0 configs/v5_fixes.rc

# Exact match (hotfixes for broken versions)
try-import-if-bazel-version ==6.3.2 configs/hotfix_6.3.2.rc

# Not equal to
try-import-if-bazel-version !=7.0.1 configs/standard.rc

# Tilde operator (range)
try-import-if-bazel-version ~1.2.3 configs/1.2.3_flags.rc
# Equivalent to >=1.2.3 <1.3.0

try-import-if-bazel-version ~1.2 configs/1.2_flags.rc
# Equivalent to >=1.2.0 <1.3.0
```

### Import Precedence

Options are interpreted in order:

```bash
# .bazelrc
build --flag1=a          # 1. Set flag1=a

import %workspace%/include.rc
# include.rc contains: build --flag1=b
# 2. flag1 is overridden to b (imported file takes precedence)

build --flag1=c          # 3. flag1 is now c (post-import takes precedence)
```

**Rule:** Options specified after the import override options in the imported file.

---

## Common Patterns

### Development Configuration

```bash
# .bazelrc

# Default: fast dev builds
build --compilation_mode=fastbuild
build --cxxopt="-std=c++17"
build --keep_going
build --cache_test_results=auto

test --test_output=short

# For slow machines
build:slow --jobs=2 --local_resources=cpu=2

# Usage: bazel build --config=slow //foo
```

### Release Configuration

```bash
# release.rc
build --compilation_mode=opt
build --cxxopt="-O3"
build --strip=always
build --stamp
test --cache_test_results=no
```

Usage in CI:
```bash
bazel build --bazelrc=/dev/null --bazelrc=release.rc //foo
```

### Per-Language Configuration

```bash
# .bazelrc

# C++
build --cxxopt="-std=c++17"
build --cxxopt="-Wall"

# Java
build --java_language_version=11

# Python
build --python_version=PY3
```

### Workspace-Local Overrides

```bash
# .bazelrc (checked in)
common --color=yes
build --compilation_mode=opt

# ~/.bazelrc.local (not checked in)
try-import ~/.bazelrc.local
# User can override by creating ~/.bazelrc.local:
# build --compilation_mode=fastbuild
```

---

## Global Bazelrc

Bazel looks for a global RC file in the install base. This file:
- Is set when Bazel is installed
- Applies to all workspaces for this user
- Can't be disabled

Use for:
- User preferences (colors, output verbosity)
- Global tool configuration

---

## Best Practices

1. **Check workspace .bazelrc into version control** — Ensures consistent builds
2. **Use .bazelrc.local for personal settings** — Add to `.gitignore`
3. **Document non-obvious flags** — Add comments explaining why
4. **Use --config for variants** — Clearer than multiple files
5. **Avoid release-specific flags in workspace .bazelrc** — Use `--bazelrc=/dev/null` in CI
6. **Keep system .bazelrc minimal** — Workspace should be authoritative
7. **Use imports for large configurations** — Split into logical files

---

## Troubleshooting

### "Option not recognized"
Check Bazel version. Some flags are version-specific. Use `try-import-if-bazel-version` for version-dependent options.

### "RC file not found"
Use `try-import` instead of `import` if the file is optional.

### "Wrong options applied"
Check RC file load order. Run `bazel info` to see configuration details.

### "Conflicts with CI configuration"
Use `--bazelrc=/dev/null` to ignore all RC files, then explicitly load needed ones.
