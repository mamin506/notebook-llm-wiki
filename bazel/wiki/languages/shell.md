---
title: "Shell Scripts in Bazel"
category: "languages"
level: "fundamentals"
status: "seedling"
sources: ["Shell Rules.md"]
tags: ["#shell", "#bash", "#scripts", "#language-specific"]
related: ["[[concepts/fundamentals/targets]]", "[[reference/general-rules]]", "[[patterns/dependency-management]]"]
last_updated: "2026-07-19"
graph-group: "languages"
---

# Shell Scripts in Bazel

Bazel's shell rules (`sh_binary`, `sh_library`, `sh_test`) allow you to package, test, and run shell scripts with proper dependency management and hermetic execution.

## Core Shell Rules

### sh_library

A reusable library of shell scripts (programs that don't require compilation).

**Key Attributes:**
- `srcs`: Shell script source files (`.sh` or any interpreter)
- `deps`: Other `sh_library` targets this library depends on
- `data`: Data files needed at runtime

**Example:**
```python
sh_library(
    name = "helpers",
    srcs = ["deploy.sh", "health_check.sh"],
)
```

**Notes:**
- `srcs`, `deps`, and `data` are mostly equivalent for shell libraries (all become runfiles)
- Use them conventionally: `srcs` for code, `deps` for libraries, `data` for data files

### sh_binary

Creates an executable shell script.

**Key Attributes:**
- `srcs`: Exactly one shell script file (must be executable)
- `deps`: Other `sh_library` targets
- `data`: Data files needed at runtime (not scripts; those go in `deps`)
- `env`: Environment variables to set at runtime
- `env_inherit`: Environment variables to inherit from the parent
- `use_bash_launcher`: Use bash launcher with runfiles support (useful for complex scripts)

**Example:**
```python
sh_binary(
    name = "deploy",
    srcs = ["deploy.sh"],
    deps = [":helpers"],
    data = glob(["config/*.yaml"]),
    env = {
        "DEPLOY_ENV": "production",
    },
)
```

**Important:**
- Script must have a shebang (e.g., `#!/bin/bash`, `#!/bin/zsh`)
- Bazel respects the shebang; any available interpreter can be used
- The rule name and script filename must be distinct

### sh_test

A shell script test runner.

**Key Attributes:**
- `srcs`: Exactly one shell script file (the test)
- `deps`: Libraries and dependencies
- `data`: Test data files
- Standard test attributes: `size`, `timeout`, `tags`, `flaky`, `shard_count`

**Example:**
```python
sh_test(
    name = "integration_test",
    srcs = ["integration_test.sh"],
    deps = [":helpers"],
    data = glob(["test_data/**/*"]),
    size = "medium",
    timeout = "short",
)
```

**Test Convention:**
- Exit code 0 = pass, non-zero = fail
- Output goes to stdout/stderr (captured by Bazel)

## Common Patterns

### Shared Utilities

Create a library of reusable shell functions:

```
scripts/
├── BUILD
├── lib.sh        # Shared functions
├── deploy.sh     # Uses lib.sh
└── health_check.sh

# BUILD:
sh_library(
    name = "lib",
    srcs = ["lib.sh"],
)

sh_binary(
    name = "deploy",
    srcs = ["deploy.sh"],
    deps = [":lib"],
)
```

Script usage:
```bash
#!/bin/bash
source "$0.runfiles/myrepo/scripts/lib.sh"
my_function "argument"
```

### Data Files in Tests

```python
sh_test(
    name = "config_test",
    srcs = ["config_test.sh"],
    data = glob(["configs/*.yaml"]),
)
```

In the test script:
```bash
#!/bin/bash
# Access data files via runfiles
CONFIG_PATH="${RUNFILES_DIR}/myrepo/configs/test.yaml"
cat "$CONFIG_PATH"
```

### Using Bash Launcher

For complex runfiles handling, enable the bash launcher:

```python
sh_binary(
    name = "app",
    srcs = ["app.sh"],
    use_bash_launcher = True,
)
```

The launcher sets up the runfiles environment automatically.

## Hermetic Scripts

Shell scripts in Bazel should be hermetic:

**Bad (non-hermetic):**
```bash
#!/bin/bash
# Assumes /usr/local/bin is in PATH
mysql --version
python3 --version
```

**Good (hermetic):**
```python
sh_binary(
    name = "app",
    srcs = ["app.sh"],
    deps = [
        "@mysql//:mysql_cli",
        "@python3//:python",
    ],
)
```

Script:
```bash
#!/bin/bash
# Use dependency paths
$MYSQL --version
$PYTHON3 --version
```

## Calling Other Targets

Run binaries from other targets:

```python
sh_binary(
    name = "deployment",
    srcs = ["deploy.sh"],
    deps = [
        ":health_check",  # Another sh_binary
        "//services:config_generator",
    ],
)
```

Script:
```bash
#!/bin/bash
RUNFILES_DIR="${0}.runfiles"
"$RUNFILES_DIR/myrepo/deployment/health_check" --check_db
"$RUNFILES_DIR/myrepo/services/config_generator" > config.yaml
```

## Build Flags

- `--action_env=VAR=value`: Pass environment variables to scripts
- `--test_arg=--verbose`: Pass arguments to test scripts
- `--test_output=all`: Show test stdout/stderr

## Common Issues

**Issue: "sh_binary rule name and script filename must be distinct"**
- Solution: Use different names
  ```python
  # Script: deploy.sh
  sh_binary(name = "deploy_tool", srcs = ["deploy.sh"])
  ```

**Issue: "permission denied" when running script**
- Script must be marked executable in source control
  ```
  chmod +x myrepo/scripts/app.sh
  git add myrepo/scripts/app.sh
  ```

**Issue: "Cannot find data files at runtime"**
- Data files are available in `${RUNFILES_DIR}` (if using bash launcher) or via `$0.runfiles/`
- Make sure files are in the `data` attribute, not `srcs` or `deps`

**Issue: "Script runs locally but fails with RBE"**
- Check for system dependencies (e.g., `/bin/bash` path assumptions)
- Declare all tools as dependencies
- Use relative paths, not absolute paths

## See Also

- [[patterns/dependency-management]] — Managing script dependencies
- [[reference/general-rules]] — Utility rules for grouping scripts
- [[languages/python]], [[languages/cpp]] — Other language guides
