---
title: "Visibility: Access Control"
category: "patterns"
level: "intermediate"
status: "growing"
sources: ["Visibility    Bazel.md"]
tags: ["visibility", "access-control", "api"]
graph-group: "patterns"
related: ["[[concepts/fundamentals/targets]]", "[[concepts/fundamentals/packages]]"]
last_updated: "2026-07-19"
---

# Visibility: Controlling Target Access

**Visibility** controls which targets may depend on your target.

A target fails to build if it depends on a target that doesn't grant it visibility.

## Visibility Specifications

Add `visibility` attribute to targets:

```starlark
cc_library(
    name = "lib",
    visibility = ["//specific/package:__pkg__"],
)
```

### Visibility Types

- `"//visibility:public"` — Accessible from anywhere
- `"//visibility:private"` — Only package-local (default if not specified)
- `"//foo/bar:__pkg__"` — Only package `//foo/bar`
- `"//foo/bar:__subpackages__"` — Package and all subpackages
- `"//some/pkg:my_group"` — Custom package group

## Default Visibility

Set package-wide default:

```starlark
package(default_visibility = ["//friend:__pkg__"])

cc_library(name = "lib1", ...)  # Inherits default + own package
cc_library(name = "lib2", visibility = ["//visibility:public"])  # Overrides
```

**Default when not specified:** `["//visibility:private"]`

## Best Practices

1. **Avoid public by default** — Explicit is better than implicit
   - Don't set `default_visibility = ["//visibility:public"]`
   - Only mark truly public API as public

2. **Use `__subpackages__` for teams** — Avoids visibility churn
   - Prefer: `"//other_team:__subpackages__"`
   - Avoid: `"//other_team:__pkg__"` (too restrictive)

3. **Use `package_group` for shared patterns** — Reduces duplication

   ```starlark
   package_group(
       name = "internals",
       packages = ["//server/...", "//cli/..."],
   )
   
   cc_library(
       name = "internal_lib",
       visibility = [":internals"],
   )
   ```

4. **Document public APIs** — What's visible should be intentional

## Example

```starlark
# //server/lib/BUILD

package(default_visibility = ["//server:__subpackages__"])

package_group(
    name = "public_api",
    packages = ["//cli", "//tests"],
)

cc_library(
    name = "internal_lib",
    # No visibility specified; uses default
    # Only visible to //server/...
)

cc_library(
    name = "public_lib",
    visibility = [":public_api"],
    # Visible to //cli and //tests, plus //server/...
)
```

---

See also: [[concepts/fundamentals/targets]], [[concepts/fundamentals/packages]]
