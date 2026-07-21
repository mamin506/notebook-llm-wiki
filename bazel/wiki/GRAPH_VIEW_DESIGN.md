# Graph View Design & Color Groups

## Overview

The wiki uses Obsidian's **Graph View color groups** to visually distinguish different types of knowledge. Each directory has a unique color, making it easy to see how concepts, tools, patterns, and reference material interconnect.

## Color Scheme

| Directory | Color | Meaning | RGB Value |
|-----------|-------|---------|-----------|
| **concepts/fundamentals/** | 🔵 Blue | Beginner-level core concepts | #3C5AA6 |
| **concepts/advanced/** | 🟦 Indigo | Expert-level advanced topics | #65469E |
| **reference/** | 🟩 Green | Official docs, API reference | #10A981 |
| **languages/** | 🟪 Purple | Language-specific guides | #A355F7 |
| **tools/** | 🟧 Orange | Ecosystem tools & utilities | #F97316 |
| **patterns/** | 🟨 Amber | Best practices & patterns | #FBBF24 |
| **troubleshooting/** | 🟥 Red | Debugging & problem-solving | #EF4444 |
| **experiments/** | 🟩 Pink | Hands-on learning & projects | #EC4899 |

## How It Works

### Configuration

The Graph View configuration is stored in `.obsidian/graph.json`:

```json
{
  "colorGroups": [
    {
      "query": "path:concepts/fundamentals",
      "color": { "a": 1, "rgb": 3949734 }
    },
    // ... other groups ...
  ]
}
```

Each group:
- Uses a `path:` query to match pages in a specific directory
- Assigns a unique RGB color
- Displays that color in the Graph View

### How to Use

1. **Open Graph View** — In Obsidian, go to View → Graph View (or Ctrl/Cmd+Shift+G)
2. **See the colors** — Each page now displays in its category's color
3. **Explore relationships** — Follow connections between concepts, patterns, tools, etc.

## Benefits

- 🎨 **Visual hierarchy** — Instantly see which pages are foundational vs. advanced
- 🔗 **Connection patterns** — Observe how different domains relate
- 📚 **Navigation aid** — Colors make it easier to scan the wiki
- 🧠 **Learning paths** — See how fundamentals → patterns → experiments flow
- 🛠️ **Tool ecosystem** — Tools visually cluster with their related concepts

## Adding Pages

When creating a new wiki page:

1. **Choose the right directory** — Put it in `concepts/`, `tools/`, `patterns/`, etc.
2. **Add frontmatter** — Include the standard frontmatter fields
3. **Graph View updates automatically** — No manual color assignment needed
4. **Optionally add tags** — Use `graph-group` tag in frontmatter for documentation

### Example Frontmatter

```yaml
---
title: "Packages: The Unit of Organization"
category: "concepts"
level: "fundamentals"
status: "growing"
sources: ["source.md"]
tags: ["core", "package", "organization"]
graph-group: "concepts-fundamentals"
related: [...]
last_updated: "2026-07-19"
---
```

## Interpretation Guide

### What Colors Mean

- **Blue (Fundamentals)** — Start here if new to Bazel
- **Indigo (Advanced)** — Deeper topics for experienced users
- **Green (Reference)** — Formal docs and API reference
- **Purple (Languages)** — Language-specific implementation details
- **Orange (Tools)** — Helper utilities in the Bazel ecosystem
- **Amber (Patterns)** — Proven best practices and design approaches
- **Red (Troubleshooting)** — Problem-solving and debugging
- **Pink (Experiments)** — Learning by doing; hands-on projects

### Exploring the Graph

1. **Follow blue nodes** to understand core Bazel concepts
2. **Branch to purple nodes** for your language of interest
3. **Link to amber nodes** for best practices
4. **Refer to green nodes** for authoritative documentation
5. **Use orange nodes** to discover helpful tools
6. **Red nodes** help when you get stuck
7. **Pink nodes** show real-world experimentation

## Future Enhancements

- Add `graph-group` tags to all existing pages (currently in frontmatter for documentation)
- Create "Graph View guided tours" (curated paths through colors)
- Add annotations to explain color relationships
- Create a print-friendly "color legend" poster

---

**Last Updated:** 2026-07-19  
**Status:** Active  
**Maintenance:** Automatic (no manual color assignment needed)
