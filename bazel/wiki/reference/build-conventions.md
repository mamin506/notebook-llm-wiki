---
title: "BUILD File Conventions & Formatting"
category: "reference"
level: "fundamentals"
status: "growing"
sources: ["BUILD Style Guide.md"]
tags: ["conventions", "formatting", "reference"]
related: ["[[patterns/build-file-style]]", "[[tools/buildifier]]"]
last_updated: "2026-07-19"
---

# BUILD File Conventions & Formatting

## Automatic Formatting with Buildifier

All BUILD files must be formatted with **Buildifier**, a standardized formatting tool similar to Go's `gofmt`.

**Why:** Ensures consistency across all BUILD files and removes formatting as a code review concern.

```bash
buildifier BUILD  # Formats a BUILD file
```

## String & Label Conventions

### Prefer Literal Strings

Avoid string concatenation or formatting. Use explicit, literal strings.

**Why:**
- Easier to read at a glance
- Automated tools (buildozer, Code Search) can find and update values
- Readability is more important than avoiding repetition in BUILD files

**Anti-Pattern:**
```starlark
NAME = "foo"
PACKAGE = "//a/b"

proto_library(
    name = "%s_proto" % NAME,
    deps = [PACKAGE + ":other_proto"],
)
```

**Better:**
```starlark
proto_library(
    name = "foo_proto",
    deps = ["//a/b:other_proto"],
)
```

### Labels Never Split

Labels should **always be string literals**, never split or concatenated.

```starlark
# GOOD: Single-line labels
deps = ["//surprisingly/long/chain/of/package/names:extravagantly_long_target_name"]

# BAD: Never split labels
deps = ["//surprisingly/long/chain/of/package/names:" +
        "extravagantly_long_target_name"]
```

Note: Buildifier automatically fuses split labels when it detects them.

## Variable & Constant Naming

- **Constants** (global, read-only): `UPPERCASE_WITH_UNDERSCORES`
  - Example: `COPTS = ["-DVERSION=5"]`
- **Variables** (local scope): `lowercase_with_underscores`

## Boolean Attributes

Use boolean values (`True`/`False`), not integers (`1`/`0`).

**Why:** `flaky = 1` could mean "rerun once", but `flaky = True` is unambiguous.

```starlark
# GOOD
py_test(
    name = "test",
    flaky = True,
)

# BAD (discouraged)
py_test(
    name = "test",
    flaky = 1,
)
```

## Quotes

Use **double quotation marks** for all strings (convention, not Python-specific).

```starlark
name = "my_target"  # GOOD
name = 'my_target'  # BAD
```

## Line Length

**No strict line length limit.** Long labels and generated BUILD files naturally exceed 79 characters.

Recommendation: Prefer readability over line breaking.

## Whitespace in Keywords

Use **spaces around `=`** in keyword arguments.

```starlark
# GOOD
cc_library(name = "foo", srcs = ["foo.cc"])

# Differs from Python style for readability
```

## Top-Level Spacing

Use **single blank line** between top-level definitions (unlike Python's two blank lines).

```starlark
cc_library(name = "a", ...)

cc_library(name = "b", ...)  # One blank line between
```

---

See also: [[tools/buildifier]]
