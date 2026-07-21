---
title: "Extra Actions Rules (Deprecated)"
category: "reference"
level: "advanced"
status: "seedling"
sources: ["Extra Actions Rules.md"]
tags: ["deprecated", "rules", "advanced", "legacy"]
related: ["[[reference/general-rules]]"]
last_updated: "2026-07-20"
---

# Extra Actions Rules (Deprecated)

⚠️ **WARNING:** Extra actions are **deprecated**. Use [[concepts/advanced/aspects]] instead.

---

## What Were Extra Actions?

Extra actions were a mechanism to insert additional build actions into Bazel's action graph, shadowing (wrapping) existing compile/link actions.

**Historical use cases:**
- Code indexing (extract metadata while compiling)
- Cross-compilation configuration
- Build analytics (track which files compiled)
- Custom tool integration (run tool alongside compiler)

---

## Why Deprecated?

Bazel introduced **aspects** as a more powerful, cleaner alternative:

| Feature | Extra Actions | Aspects |
|---------|---------------|---------|
| **Wrap existing actions** | ✅ Yes | ✅ Yes (better) |
| **Access action inputs** | ✅ Limited | ✅ Full |
| **Composability** | ❌ No | ✅ Yes |
| **Error handling** | ⚠️ Poor | ✅ Good |
| **Language support** | ❌ Limited | ✅ Full |

**Aspects are superior** in every way. Bazel team recommends migrating all extra actions to aspects.

---

## Rules (for Reference Only)

### action_listener

Maps action mnemonics to extra actions to be injected.

```starlark
action_listener(
    name = "index_all_languages",
    mnemonics = [
        "Javac",
        "CppCompile",
        "Python",
    ],
    extra_actions = [":indexer"],
)
```

**Key attributes:**
- `mnemonics` — Action types to listen for (e.g., "Javac", "CppCompile")
- `extra_actions` — Targets to inject for each matched action

**Enable:** `bazel build --experimental_action_listener=//path:listener //...`

**Problem:** Mnemonics are not a stable public API; they change between Bazel versions.

### extra_action

The actual action to inject (deprecated rule).

```starlark
extra_action(
    name = "indexer",
    tools = ["//my/tools:indexer"],
    cmd = "$(location //my/tools:indexer) --extra_action_file=$(EXTRA_ACTION_FILE)",
)
```

**Key attributes:**
- `cmd` — Command to run
- `tools` — Tool dependencies
- `requires_action_output` — Whether original action's output is needed
- `out_templates` — Output file patterns

---

## Migration Path: Extra Actions → Aspects

### Before (Extra Actions - Deprecated)

```starlark
# BUILD
action_listener(
    name = "indexer",
    mnemonics = ["Javac"],
    extra_actions = [":index_action"],
)

extra_action(
    name = "index_action",
    tools = ["//tools:indexer"],
    cmd = "$(location //tools:indexer) --extra_action_file=$(EXTRA_ACTION_FILE)",
)

# Usage:
# bazel build --experimental_action_listener=//path:indexer //...
```

### After (Aspects - Modern)

```starlark
# BUILD
aspect(
    name = "indexer_aspect",
    attr_aspects = ["deps"],
    implementation = _indexer_impl,
)

def _indexer_impl(target, ctx):
    # Access target attributes directly
    # Run indexing on all language targets
    # Return results
    ...

# Usage:
# bazel build --aspects=//path:indexer.indexer_aspect //...
```

**Benefits of aspects:**
- ✅ Direct access to target attributes
- ✅ Can be composed (multiple aspects on one target)
- ✅ Better error handling
- ✅ Standard feature (not experimental)

---

## Current Recommendations

**For new code:** Always use aspects instead of extra actions.

**For existing code:**
1. Evaluate if functionality is still needed
2. If yes, migrate to aspects
3. Remove extra action rules

**Why now?**
- Aspects are stable and well-tested
- Extra actions maintenance burden decreasing
- Deprecation allows Bazel internals to evolve

---

## See Also

- [[concepts/advanced/aspects]] — The modern replacement (when created)
- [[reference/general-rules]] — Other general-purpose rules
