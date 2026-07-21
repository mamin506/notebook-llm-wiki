# Bazel LLM Wiki Initialization Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Initialize a persistent, compounding Bazel learning knowledge base with folder structure, schema, workflows, and configuration files.

**Architecture:** Three-layer architecture (raw sources, wiki, schema) with LLM-maintained markdown files organized by category and learning level. Workflows (ingest, query, lint) are documented as custom Obsidian skills in `.claude/skills/`.

**Tech Stack:** Markdown, YAML frontmatter, Obsidian vault, Git

## Global Constraints

- Vault root: `D:\github\notebook-llm-wiki\bazel`
- All paths are relative to vault root unless otherwise specified
- Markdown files use YAML frontmatter (YAML 1.1)
- Wiki pages use wikilinks `[[path/to/page]]` for internal cross-references
- Folder structure must match exactly as specified in design spec section 3-4
- All commits use `Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>` trailer

---

## File Structure

```
bazel/
├── raw/
│   ├── articles/              [Empty directory for web articles]
│   ├── docs/                  [Empty directory for official docs]
│   ├── books/                 [Empty directory for book content]
│   ├── videos/                [Empty directory for transcripts]
│   ├── experiments/           [Empty directory for user experiments]
│   └── assets/                [Empty directory for images/diagrams]
│
├── wiki/
│   ├── concepts/
│   │   ├── fundamentals/      [Empty, will hold beginner concepts]
│   │   └── advanced/          [Empty, will hold expert concepts]
│   ├── reference/             [Empty, will hold reference pages]
│   ├── languages/             [Empty, will hold language guides]
│   ├── tools/                 [Empty, will hold tool guides]
│   ├── patterns/              [Empty, will hold patterns/best practices]
│   ├── troubleshooting/       [Empty, will hold debugging/issues]
│   ├── experiments/           [Empty, will hold user experiments]
│   ├── index.md              [Stub: wiki catalog]
│   └── log.md                [Stub: activity log]
│
├── .claude/
│   ├── CLAUDE.md             [Master schema and instructions]
│   └── skills/
│       ├── ingest.md         [Ingest workflow skill]
│       ├── query.md          [Query workflow skill]
│       └── lint.md           [Lint workflow skill]
│
├── .gitignore                [Git ignore rules for raw/]
├── docs/
│   └── superpowers/
│       └── specs/
│           └── 2026-07-17-bazel-llm-wiki-design.md [Already committed]
│       └── plans/
│           └── 2026-07-17-bazel-llm-wiki-init.md [This file]
│
├── .obsidian/                [Existing]
└── .claudian/                [Existing]
```

---

## Task 1: Create Raw Sources Folder Structure

**Files:**
- Create: `raw/articles/` (directory)
- Create: `raw/docs/` (directory)
- Create: `raw/books/` (directory)
- Create: `raw/videos/` (directory)
- Create: `raw/experiments/` (directory)
- Create: `raw/assets/` (directory)

**Interfaces:**
- Produces: Directory structure for storing immutable source documents

---

- [ ] **Step 1: Create raw/ subdirectories**

Run these commands from vault root:

```bash
mkdir -p raw/articles raw/docs raw/books raw/videos raw/experiments raw/assets
```

- [ ] **Step 2: Verify directory structure**

```bash
ls -R raw/
```

Expected output:
```
raw/:
articles  assets  books  docs  experiments  videos

raw/articles:

raw/assets:

raw/books:

raw/docs:

raw/experiments:

raw/videos:
```

- [ ] **Step 3: Commit**

```bash
git add raw/
git commit -m "chore: initialize raw sources directory structure

Create subdirectories for articles, docs, books, videos, experiments, and assets.
These directories will hold immutable source documents.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Task 2: Create Wiki Folder Structure

**Files:**
- Create: `wiki/concepts/fundamentals/` (directory)
- Create: `wiki/concepts/advanced/` (directory)
- Create: `wiki/reference/` (directory)
- Create: `wiki/languages/` (directory)
- Create: `wiki/tools/` (directory)
- Create: `wiki/patterns/` (directory)
- Create: `wiki/troubleshooting/` (directory)
- Create: `wiki/experiments/` (directory)

**Interfaces:**
- Produces: Directory structure for LLM-maintained wiki organized by category and learning level

---

- [ ] **Step 1: Create wiki/ subdirectories**

```bash
mkdir -p wiki/concepts/fundamentals wiki/concepts/advanced wiki/reference wiki/languages wiki/tools wiki/patterns wiki/troubleshooting wiki/experiments
```

- [ ] **Step 2: Verify directory structure**

```bash
find wiki -type d
```

Expected output (8 directories total):
```
wiki
wiki/concepts
wiki/concepts/fundamentals
wiki/concepts/advanced
wiki/reference
wiki/languages
wiki/tools
wiki/patterns
wiki/troubleshooting
wiki/experiments
```

- [ ] **Step 3: Commit**

```bash
git add wiki/
git commit -m "chore: initialize wiki directory structure

Create folders for concepts (fundamentals/advanced), reference, languages,
tools, patterns, troubleshooting, and experiments. LLM will populate these
with markdown pages organized by category and learning level.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Task 3: Create .claude/skills Directory

**Files:**
- Create: `.claude/skills/` (directory)

**Interfaces:**
- Produces: Directory for custom workflow skills

---

- [ ] **Step 1: Create skills directory**

```bash
mkdir -p .claude/skills
```

- [ ] **Step 2: Verify directory exists**

```bash
ls -la .claude/
```

Expected output includes:
```
drwxr-xr-x ... skills
```

- [ ] **Step 3: Commit**

```bash
git add .claude/
git commit -m "chore: initialize .claude/skills directory

Create skills directory for custom Obsidian skills (ingest, query, lint).

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Task 4: Write CLAUDE.md (Master Schema)

**Files:**
- Create: `.claude/CLAUDE.md`

**Interfaces:**
- Produces: Master configuration file with:
  - Core principles
  - Folder structure overview
  - Frontmatter conventions
  - Workflow summaries (ingest, query, lint)
  - Maintenance discipline
  - References to custom skills

---

- [ ] **Step 1: Create CLAUDE.md with header and core principles**

Create file `.claude/CLAUDE.md`:

```markdown
# Bazel LLM Wiki Schema

**Vault Purpose:** Persistent, compounding knowledge base for Bazel learning (fundamentals through mastery)

**Philosophy:** Unlike RAG systems that re-derive knowledge on every query, this wiki *accumulates* knowledge. Cross-references are built in. Contradictions are flagged. Synthesis reflects everything learned so far. The wiki compounds with every ingested source and query explored.

**Design Spec:** See `docs/superpowers/specs/2026-07-17-bazel-llm-wiki-design.md` for complete architectural details.

---

## Three-Layer Architecture

### Layer 1: Raw Sources (`raw/`)

Immutable collection of source documents. The LLM reads from here but never modifies.

```
raw/
├── articles/        # Web articles, blog posts (clipped via Web Clipper)
├── docs/           # Official Bazel documentation, guides
├── books/          # Book chapters, long-form content
├── videos/         # Video transcripts
├── experiments/    # Your own BUILD files, test projects
└── assets/         # Images, diagrams extracted from sources
```

**Principle:** Raw sources are your source of truth. All synthesis, cross-referencing, and organization happens in the wiki layer.

### Layer 2: Wiki (`wiki/`)

LLM-generated and maintained knowledge base, organized by category and learning level.

```
wiki/
├── concepts/              # Core Bazel ideas
│   ├── fundamentals/      # Beginner-level concepts
│   └── advanced/          # Expert-level concepts
├── reference/             # Synthesis of official docs
├── languages/             # Language-specific guides
├── tools/                 # Ecosystem tools
├── patterns/              # Best practices & design patterns
├── troubleshooting/       # Debugging & performance
├── experiments/           # Pages from YOUR hands-on work
├── index.md              # Wiki catalog (maintained by LLM)
└── log.md                # Activity timeline (append-only)
```

### Layer 3: Schema (`.claude/`)

Configuration and workflow instructions.

```
.claude/
├── CLAUDE.md             # This file: core schema
└── skills/
    ├── ingest.md         # Workflow: add new sources
    ├── query.md          # Workflow: ask questions
    └── lint.md           # Workflow: health checks
```

---

## Frontmatter Schema

Every wiki page opens with YAML frontmatter:

```yaml
---
title: "Page Title"
category: "concepts|reference|languages|tools|patterns|troubleshooting|experiments"
level: "fundamentals|intermediate|advanced"
status: "seedling|growing|mature|complete"
sources: ["source-file-1.md", "source-file-2.md"]
tags: ["tag1", "tag2"]
related: ["[[other-page]]", "[[another-page]]"]
last_updated: "YYYY-MM-DD"
---
```

**Field Definitions:**

| Field | Purpose | Values |
|-------|---------|--------|
| `title` | Page title | String |
| `category` | Wiki section | concepts, reference, languages, tools, patterns, troubleshooting, experiments |
| `level` | Learning progression | fundamentals, intermediate, advanced |
| `status` | Completion state | seedling, growing, mature, complete |
| `sources` | Traceability | List of raw file names |
| `tags` | Cross-cutting themes | List of tags (#monorepo, #performance, #testing, etc.) |
| `related` | Cross-references | List of wikilinks to related pages |
| `last_updated` | Freshness tracking | ISO date (YYYY-MM-DD) |

**Status Meanings:**
- **seedling** — Just created from one source, minimal content
- **growing** — Multiple sources integrated, expanding
- **mature** — Well-developed, comprehensive, ready for reference
- **complete** — Expert-level coverage, unlikely to change

---

## Workflows

### INGEST: Adding New Sources

**Trigger:** You add a new file to `raw/` or ask to process a source.

**Workflow:** (See `.claude/skills/ingest.md` for detailed instructions)

1. Read & analyze the source
2. Create/update wiki pages for each concept
3. Add wikilinks and cross-references
4. Update `wiki/index.md`
5. Append entry to `wiki/log.md`

**Expected Output:** 1–10 new/updated wiki pages + index update + log entry

### QUERY: Asking Questions

**Trigger:** You ask a question about Bazel.

**Workflow:** (See `.claude/skills/query.md` for detailed instructions)

1. Search `wiki/index.md` for relevant pages
2. Read identified pages
3. Synthesize answer with citations
4. File valuable results back into wiki (if applicable)
5. Append entry to `wiki/log.md`

**Expected Output:** Answer + possibly new synthesis page + log entry

### LINT: Health Checks

**Trigger:** Monthly or after major ingests.

**Workflow:** (See `.claude/skills/lint.md` for detailed instructions)

1. Check for contradictions
2. Spot stale claims
3. Find orphan pages
4. Identify gaps
5. Check cross-references
6. Scan for incomplete sections
7. Log the lint pass

**Expected Output:** Health report + fixed issues + log entry + suggested follow-up sources

---

## Frontmatter Discipline

### Keep Status Current

Progression example:
- Day 1: Create page as "seedling" from one source
- Day 5: Ingest another source, update to "growing"
- Day 20: Ingest video, refine explanation, update to "mature"
- Day 60: Lint pass confirms coverage, mark "complete"

### Update last_updated on Every Edit

- Update `last_updated` field whenever you edit a page
- Enables lint to spot stale content (> 3 months old)
- Format: ISO date (YYYY-MM-DD)

### Maintain Related Links

- When creating a page, immediately add `related` entries
- Bidirectional: if A → B, then B → A
- These links ARE the knowledge base; keep them accurate

### Use Tags for Cross-Cutting Themes

Examples:
- `#monorepo` — Monorepo architecture (concepts, patterns, languages)
- `#performance` — Build performance (troubleshooting, patterns)
- `#testing` — Testing strategy (patterns, languages)
- `#debugging` — Debugging techniques (troubleshooting)
- `#external-deps` — External dependencies (patterns, troubleshooting)

### Source Traceability

Always track which raw files contributed to a page:

```yaml
sources: ["docs/concepts-guide.md", "blog-post-macros.md"]
```

Enables:
- Impact analysis ("if this source is wrong, which pages are affected?")
- Prioritization ("which pages need updates based on newest sources?")
- Credibility ("what's the evidence for this claim?")

---

## Index and Log Conventions

### wiki/index.md

Searchable catalog of all wiki pages, organized by category.

**Format:**
```markdown
# Bazel Wiki Index

## Concepts

### Fundamentals
- [[concepts/fundamentals/targets]] — What targets are (seedling)
- [[concepts/fundamentals/rules]] — What rules are (growing)

### Advanced
- [[concepts/advanced/macros]] — DSL abstractions (seedling)

## Reference
- [[reference/cli-reference]] — Bazel CLI (mature)
...
```

**Maintenance:** Update after every ingest (add pages, update status).

### wiki/log.md

Append-only timeline of ingests, queries, and lint passes.

**Format:**
```markdown
# Bazel Wiki Activity Log

## [2026-07-17] ingest | Official Bazel Concepts Guide

- Created: [[concepts/fundamentals/targets]], [[concepts/fundamentals/rules]]
- Updated: [[reference/api-guide]]
- Key findings: [Summary]

## [2026-07-18] query | How do macros differ from rules?

- Pages referenced: [[concepts/fundamentals/rules]]
- Answer: [Summary]
- New page: [[concepts/advanced/macros-vs-rules]]
...
```

**Convention:** Each entry starts with `## [YYYY-MM-DD]` so it's grep-able: `grep "^## \[" log.md`

---

## Usage Tips

1. **Before ingesting:** Read `{{.claude/skills/ingest.md}}` for detailed workflow
2. **Before querying:** Use the wiki as-is; search index first
3. **Monthly maintenance:** Run lint pass (see `.claude/skills/lint.md`)
4. **Obsidian graph view:** Shows the shape of your wiki; watch for orphans
5. **Dataview plugin (optional):** Can query frontmatter for dynamic tables

---

## Next Steps

1. Add first source: Official Bazel documentation
2. Create foundational pages in `concepts/fundamentals/`
3. Populate `reference/` with synthesis
4. Iterate: ingest → query → lint

See `docs/superpowers/specs/2026-07-17-bazel-llm-wiki-design.md` for complete architecture details.
```

- [ ] **Step 2: Verify CLAUDE.md was created**

```bash
test -f .claude/CLAUDE.md && echo "CLAUDE.md exists" || echo "Error: CLAUDE.md not found"
```

Expected: `CLAUDE.md exists`

- [ ] **Step 3: Commit**

```bash
git add .claude/CLAUDE.md
git commit -m "docs: add CLAUDE.md master schema for Bazel LLM Wiki

Documents core principles, three-layer architecture (raw sources, wiki, schema),
folder structure, frontmatter conventions, three workflows (ingest/query/lint),
and maintenance discipline.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Task 5: Write .claude/skills/ingest.md (Ingest Workflow)

**Files:**
- Create: `.claude/skills/ingest.md`

**Interfaces:**
- Consumes: Design spec (section 6.1) and frontmatter schema (CLAUDE.md)
- Produces: Callable skill documenting ingest workflow with step-by-step procedure, checklist, and examples

---

- [ ] **Step 1: Create ingest.md skill file**

Create file `.claude/skills/ingest.md`:

```markdown
# Ingest Skill: Adding New Sources to the Wiki

**Purpose:** Process a new source document and integrate it into the wiki.

**When to use:** When you add a file to `raw/` or ask the LLM to process a source.

**Expected time:** 15–30 minutes per source (depending on length and complexity)

---

## Workflow

### 1. Read & Analyze

- [ ] Read the entire source carefully
- [ ] Identify key concepts, principles, examples
- [ ] Note contradictions with existing wiki pages
- [ ] Note gaps (things mentioned but not explained)
- [ ] List all new or updated topics

**Output:** Mental model of source's main ideas and how they relate to existing wiki

### 2. Create/Update Wiki Pages

For each concept identified:

- [ ] Does a wiki page exist? (Search `wiki/index.md`)
  - **If yes:** Read existing page
    - Integrate new information into existing page
    - Add examples from source
    - Flag contradictions in page (note which source caused discrepancy)
    - Update `status` field (seedling → growing, or growing → mature)
  - **If no:** Create new page with:
    - Appropriate `category` (concepts, reference, languages, tools, patterns, troubleshooting, or experiments)
    - Appropriate `level` (fundamentals, intermediate, or advanced)
    - `status: seedling` (just created from one source)
    - Complete frontmatter as per schema
    - Content extracted from source

- [ ] Update all affected pages' frontmatter:
  - Add source filename to `sources:` list (if not already there)
  - Update `status` field
  - Update `last_updated: "YYYY-MM-DD"`

**Output:** 1–10 new or updated markdown pages with complete frontmatter

### 3. Update Cross-References

- [ ] Review all created/updated pages
- [ ] For each page, identify related concepts in the wiki
- [ ] Add `related:` wikilinks in both directions
  - If page A links to page B, then page B should link to page A
  - Example: if `concepts/fundamentals/targets.md` links to `concepts/fundamentals/rules.md`, then rules should link back to targets

- [ ] Update `related:` lists to include newly discovered connections

**Output:** All cross-references in both directions; no orphan pages from this ingest

### 4. Update Index

- [ ] Open `wiki/index.md`
- [ ] Add new pages to appropriate section (organized by category, then by level)
- [ ] Update status indicators for modified pages
- [ ] Keep format consistent: `- [[path/to/page]] — Short description (status)`
- [ ] Ensure all categories are represented

**Output:** `wiki/index.md` is current and complete

### 5. Log the Ingest

- [ ] Open `wiki/log.md`
- [ ] Append new entry at end:

```markdown
## [YYYY-MM-DD] ingest | [Source Title]

- Created: [[page1]], [[page2]]
- Updated: [[page3]], [[page4]]
- Key findings: [2-3 sentence summary of main insights]
- Open questions: [Questions raised but not yet answered]
- Next source: [Suggestion for follow-up reading]
```

- [ ] Verify format: entry starts with `## [YYYY-MM-DD]` so it's grep-able

**Output:** New log entry appended

---

## Verification Checklist

Before completing ingest, verify:

- [ ] All new pages have complete YAML frontmatter
- [ ] All pages' `sources:` field lists contributing source files
- [ ] All pages' `status` reflects current maturity (seedling/growing/mature/complete)
- [ ] All pages' `last_updated` is today's date (YYYY-MM-DD)
- [ ] All `related:` links are bidirectional (if A→B, then B→A)
- [ ] No orphan pages (all new pages linked from at least one other page)
- [ ] `wiki/index.md` is updated with all new/modified pages
- [ ] `wiki/log.md` has new entry with correct format `## [YYYY-MM-DD]`

---

## Bazel-Specific Tips

1. **Terminology:** Use Bazel's exact terminology (e.g., "target", "rule", "artifact", not "build rule" or "output")
2. **Examples:** Include concrete BUILD file examples when explaining concepts
3. **Cross-domain:** A source about Python might mention Bazel rules — file relevant insights in both `languages/python.md` and `concepts/` as appropriate
4. **Contradictions:** When sources disagree, note the discrepancy in the page (don't hide it): "Source A says X, but Source B says Y; [explanation]"
5. **Official docs first:** Official Bazel docs are more authoritative than blog posts; if they disagree, official is correct (but still note both)

---

## Example: Ingesting the Official Concepts Guide

**Source:** `raw/docs/bazel-concepts-guide.md`

**Result:**

Created:
- `concepts/fundamentals/targets.md` (seedling)
- `concepts/fundamentals/rules.md` (seedling)
- `concepts/fundamentals/artifacts.md` (seedling)
- `reference/api-guide.md` (seedling)

Updated:
- None (first ingest)

Cross-references added:
- targets ↔ rules ↔ artifacts (all linked)
- reference/api-guide linked from all concept pages

Log entry:
```
## [2026-07-17] ingest | Official Bazel Concepts Guide

- Created: [[concepts/fundamentals/targets]], [[concepts/fundamentals/rules]], [[concepts/fundamentals/artifacts]], [[reference/api-guide]]
- Updated: None
- Key findings: Targets are immutable atomic units; rules are parameterized build instructions; artifacts are outputs of rules
- Open questions: How do we express complex dependencies? How do workspaces relate to packages?
- Next source: Find a guide on dependency graphs and workspace configuration
```

---

## Common Pitfalls

1. **Isolated pages:** Creating pages without linking them → lint finds orphans. Always add `related:` immediately.
2. **Stale sources:** Not updating `sources:` field → traceability lost. Always list contributing sources.
3. **Inconsistent terminology:** Using "rule" in one page, "build rule" in another. Use Bazel's exact terminology.
4. **Skipping cross-references:** Only updating page A, forgetting to update page B even though they're related. Always update bidirectionally.
5. **Not updating index:** Forgetting to add new pages to `wiki/index.md` → discovery fails. Update index after every ingest.
```

- [ ] **Step 2: Verify ingest.md was created**

```bash
test -f .claude/skills/ingest.md && echo "ingest.md exists" || echo "Error: ingest.md not found"
```

Expected: `ingest.md exists`

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/ingest.md
git commit -m "docs: add ingest skill for processing new sources

Documents step-by-step workflow for ingesting sources: read/analyze,
create/update wiki pages, cross-reference, update index, log entry.
Includes verification checklist, Bazel-specific tips, and examples.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Task 6: Write .claude/skills/query.md (Query Workflow)

**Files:**
- Create: `.claude/skills/query.md`

**Interfaces:**
- Consumes: Design spec (section 6.2), index format, and frontmatter schema
- Produces: Callable skill documenting query workflow with step-by-step procedure, examples, and guidelines

---

- [ ] **Step 1: Create query.md skill file**

Create file `.claude/skills/query.md`:

```markdown
# Query Skill: Asking Questions Against the Wiki

**Purpose:** Answer questions about Bazel by synthesizing information from existing wiki pages.

**When to use:** When you ask a question about Bazel or the wiki itself.

**Expected time:** 5–15 minutes per query (depending on scope and depth)

---

## Workflow

### 1. Search the Index

- [ ] Read `wiki/index.md` from top to bottom
- [ ] Identify all pages that might be relevant to the question
- [ ] Look for:
  - Pages with matching keywords in title or description
  - Pages in relevant categories (e.g., troubleshooting for "why is my build slow")
  - Pages in related categories (e.g., languages/python for "how do I use pytest with Bazel")

**Output:** List of candidate pages to read

### 2. Read Relevant Pages

- [ ] For each candidate page:
  - [ ] Read the entire page (including frontmatter)
  - [ ] Note the source(s) that contributed to it
  - [ ] Look at `related:` links for adjacent concepts
  - [ ] Read linked pages if they add context to the answer

- [ ] Collect:
  - Direct answers to the question
  - Related concepts that clarify the answer
  - Contradictions or uncertainties
  - Gaps in coverage

**Output:** Synthesis material with citations

### 3. Synthesize & Answer

- [ ] Compose answer that:
  - Directly addresses the question
  - Cites relevant pages (e.g., "See [[concepts/fundamentals/targets]] for details")
  - Uses Bazel terminology consistently
  - Includes concrete examples if helpful
  - Calls out contradictions or uncertain areas
  - Suggests related topics for further exploration

- [ ] Format answer appropriately:
  - Simple questions → markdown paragraph
  - Complex questions → markdown with headers, tables, or lists
  - Comparisons → table or side-by-side
  - Step-by-step → numbered list

**Output:** Clear, cited answer

### 4. File if Valuable

- [ ] Evaluate: Is this answer valuable enough to keep in the wiki?
  - **Yes if:** Answer reveals new synthesis, useful comparison, or pattern worth keeping
  - **No if:** Answer simply retrieves existing information, or is too question-specific

- [ ] If yes:
  - [ ] Create new wiki page (e.g., `concepts/advanced/macros-vs-rules.md` for a comparison)
  - [ ] Add to appropriate category
  - [ ] Set `status: seedling` (just created from this query)
  - [ ] Set `sources: []` (synthesized, not from raw source; can add raw sources if any)
  - [ ] Add to `wiki/index.md`
  - [ ] Update `related:` in both directions

- [ ] If no:
  - [ ] Simply provide answer without filing

**Output:** New wiki page (if valuable) or answer-only response

### 5. Log the Query

- [ ] Open `wiki/log.md`
- [ ] Append new entry:

```markdown
## [YYYY-MM-DD] query | [Question Summary]

- Pages referenced: [[page1]], [[page2]]
- Answer: [1–2 sentence summary]
- New page created: [[new-page]] (if applicable)
- Gaps identified: [Any missing info or contradictions noted]
- Suggested follow-up source: [If applicable]
```

- [ ] Verify format: entry starts with `## [YYYY-MM-DD]`

**Output:** New log entry appended

---

## Verification Checklist

Before completing query, verify:

- [ ] Answer addresses the question directly
- [ ] All claims are cited to wiki pages
- [ ] Bazel terminology is used consistently
- [ ] If new page created:
  - [ ] Complete YAML frontmatter
  - [ ] Added to `wiki/index.md`
  - [ ] Bidirectional `related:` links updated
- [ ] `wiki/log.md` has new entry with correct format

---

## Query Examples

### Example 1: Simple Factual Question

**Question:** "What's a Bazel target?"

**Process:**
1. Search index → find `concepts/fundamentals/targets.md`
2. Read page
3. Answer: "A target is the atomic unit of a Bazel build. It specifies what to build (a rule) and how to build it (attributes). Targets are named with labels like `//myapp:service`. See [[concepts/fundamentals/targets]] for details."
4. No new page needed (answer already exists in wiki)
5. Log entry (optional for simple lookups, but recommended)

### Example 2: Synthesis Question Requiring New Page

**Question:** "When should I use macros vs. rules?"

**Process:**
1. Search index → find `concepts/fundamentals/rules.md`, `concepts/advanced/macros.md`
2. Read both pages, note they explain concepts separately but don't compare
3. Answer with synthesis (macros expand at load time, rules execute during build, use macros for DSL abstractions, use rules for build actions)
4. This synthesis is valuable → create `concepts/advanced/macros-vs-rules.md`
5. Add to index, update `related:` in both directions
6. Log entry for query + new page creation

### Example 3: Troubleshooting Question

**Question:** "Why is my build so slow?"

**Process:**
1. Search index → find pages tagged #performance, in `troubleshooting/performance.md`, `patterns/dependency-management.md`
2. Read pages for common causes
3. Answer with: "Check remote cache, verify dependencies are correct, profile with `bazel build --profile`, see [[troubleshooting/performance.md]] for detailed optimization techniques"
4. No new page (answer is in existing pages)
5. Log entry

---

## Guidelines

1. **Use the wiki first:** Always search index before asking external sources
2. **Cite liberally:** Every claim should reference a wiki page
3. **Call out gaps:** If the wiki is missing something, note it
4. **File valuable synthesis:** New comparisons, patterns, or connections belong in the wiki
5. **Keep learning:** Questions often suggest follow-up sources to ingest
```

- [ ] **Step 2: Verify query.md was created**

```bash
test -f .claude/skills/query.md && echo "query.md exists" || echo "Error: query.md not found"
```

Expected: `query.md exists`

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/query.md
git commit -m "docs: add query skill for answering questions against the wiki

Documents step-by-step workflow for querying wiki: search index, read pages,
synthesize answer, file if valuable, log entry. Includes verification checklist,
examples, and guidelines.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Task 7: Write .claude/skills/lint.md (Lint Workflow)

**Files:**
- Create: `.claude/skills/lint.md`

**Interfaces:**
- Consumes: Design spec (section 6.3) and wiki structure
- Produces: Callable skill documenting lint workflow with step-by-step procedure, checks, and examples

---

- [ ] **Step 1: Create lint.md skill file**

Create file `.claude/skills/lint.md`:

```markdown
# Lint Skill: Periodic Wiki Health Checks

**Purpose:** Maintain wiki health by finding and fixing contradictions, gaps, orphans, and stale content.

**When to use:** Monthly or after major ingest batches. Can be requested anytime.

**Expected time:** 30–60 minutes for a comprehensive lint pass (depends on wiki size)

---

## Workflow

### 1. Check for Contradictions

- [ ] Skim all pages in the wiki (use index.md as guide)
- [ ] Look for pages that claim opposite things
- [ ] Note:
  - Which pages contradict
  - What the claims are
  - Which source(s) caused the conflict

**Resolution Options:**
- [ ] Update one page with newer/more accurate information
- [ ] Note discrepancy in both pages if both are valid (e.g., "older Bazel versions did X, newer versions do Y")
- [ ] Suggest a new source to clarify

**Output:** List of contradictions found and resolved

### 2. Spot Stale Claims

- [ ] Review pages with `last_updated` > 3 months ago
- [ ] Check if any facts are outdated by newer sources
- [ ] Look for:
  - Version-specific information (e.g., "Bazel 5.0 added X" might be outdated)
  - Best practices that may have changed
  - Deprecated features

**Resolution:**
- [ ] Update page with newer information
- [ ] Add source to `sources:` field
- [ ] Update `last_updated` to today

**Output:** List of stale claims found and updated

### 3. Find Orphan Pages

- [ ] Go through every page in wiki/
- [ ] Check: Does this page have inbound links (except from index.md)?
- [ ] Orphans are pages with:
  - No `related:` entries pointing TO this page
  - No wikilinks from other pages
  - Only entry in index.md

**Resolution Options:**
- [ ] Add cross-references from related pages
- [ ] Integrate into a parent page if it's too narrow
- [ ] Delete if truly obsolete

**Output:** List of orphan pages found and linked/integrated

### 4. Identify Gaps

- [ ] Look across all pages for important concepts mentioned but lacking their own page
- [ ] Examples:
  - "See the Bazel API reference" mentioned multiple times, but no `reference/api.md` exists
  - Python mentioned in `languages/` but no dedicated Python page
  - Performance mentioned in `troubleshooting/` but no dedicated guide

**Resolution:**
- [ ] Create new page for important concepts
- [ ] Link from all pages that reference the concept
- [ ] Set new page to `status: seedling` and note it's a gap-filler

**Output:** List of new pages created for gaps, or marked for follow-up sources

### 5. Check Cross-References

- [ ] Review all `related:` fields
- [ ] Verify bidirectionality:
  - If `A.md` has `related: [[[B]]]`, then `B.md` should have `related: [[[A]]]`

**Resolution:**
- [ ] Update missing back-references

**Output:** All cross-references now bidirectional

### 6. Scan for Incomplete Sections

- [ ] Look for pages marked "TODO", "[incomplete]", or with obvious gaps
- [ ] Note:
  - Which pages have missing sections
  - What information is missing

**Resolution:**
- [ ] Complete the section if you can
- [ ] Or, mark with note: "This needs [source type]" and add to follow-up list

**Output:** Incomplete sections completed or flagged for follow-up

### 7. Log the Lint Pass

- [ ] Open `wiki/log.md`
- [ ] Append new entry:

```markdown
## [YYYY-MM-DD] lint | [Lint Pass Type]

- Contradictions: N found (action: resolved/noted)
- Stale claims: N found (updated/verified)
- Orphan pages: N found (linked/integrated/deleted)
- Gaps: N found (new pages created: [list], marked for follow-up: [count])
- Missing cross-refs: N found (all fixed)
- Incomplete sections: N found (completed/flagged)
- Overall: [Health summary]
- Recommended follow-up sources: [List any gaps worth exploring]
```

- [ ] Verify format: entry starts with `## [YYYY-MM-DD]`

**Output:** New log entry appended

---

## Verification Checklist

Before completing lint, verify:

- [ ] All contradictions documented or resolved
- [ ] All stale pages updated with `last_updated` field
- [ ] All orphan pages either linked, integrated, or deleted
- [ ] New gap-filling pages created with complete frontmatter
- [ ] All `related:` fields are bidirectional
- [ ] All incomplete sections either completed or flagged
- [ ] `wiki/log.md` has new entry with correct format
- [ ] Suggested follow-up sources documented

---

## Lint Cadence & Triggers

### Monthly Lint (Regular Maintenance)

Run once a month to keep wiki healthy:
- Revisit last month's entries in log.md
- Check for contradictions that emerged
- Verify stale content
- Spot gaps from recent ingests

### Post-Ingest Lint

After ingesting a major source (e.g., 5+ new pages):
- Check for orphans (newly created pages should be linked)
- Verify cross-references (new pages should be bidirectional)
- Spot contradictions (new content vs. existing)

### Periodic Deep Lint

Every 3 months, run a comprehensive lint:
- Full review of all pages
- Cross-check all frontmatter fields
- Rebuild any broken cross-references
- Comprehensive gap analysis

---

## Lint Example: Post-Ingest Check

**After ingesting:** "Advanced Bazel Patterns" (10 new pages in `patterns/`)

**Lint results:**

```
## [2026-07-20] lint | Post-ingest check for patterns batch

- Contradictions: 2 found
  * patterns/monorepo-layout.md says "use 3 levels", but concepts/fundamentals/packages.md says "use 2 levels" (different contexts—updated both to clarify)
  * patterns/testing.md mentions Bazel 6.0 feature, but tools/ is older (updated reference)

- Stale claims: 1
  * languages/python.md mentions "old py_test", updated to current version

- Orphan pages: 3 new patterns pages had no inbound links
  * patterns/custom-rules.md ← linked from concepts/advanced/custom-rules.md
  * patterns/dependency-management.md ← linked from reference/api-guide.md
  * patterns/cross-platform.md ← linked from languages/cpp.md, languages/java.md

- Gaps: 3 identified
  * "Debugging custom rules" mentioned but no page—created patterns/debugging-rules.md (seedling)
  * "Performance profiling" mentioned multiple times—marked for follow-up source
  * "Integrating external repos" mentioned but unclear—marked for follow-up source

- Missing cross-refs: 5 found and added
  * patterns/monorepo-layout.md ← → concepts/fundamentals/packages.md
  * patterns/testing.md ← → troubleshooting/test-failures.md
  * tools/gazelle.md ← → patterns/monorepo-layout.md (was missing reverse link)

- Incomplete sections: 2 found
  * patterns/custom-rules.md section "Advanced: Macros" flagged as incomplete—suggested follow-up source
  * reference/api-guide.md section "Deprecated APIs" needs update—marked for next lint pass

- Overall: Wiki is healthy. New patterns batch well-integrated. 3 gaps identified, 2 stale claims fixed.

- Recommended follow-up sources: Performance profiling guide, external repo integration guide, debugging custom rules tutorial
```

---

## Lint Tips

1. **Use grep:** `grep "^- " wiki/index.md | wc -l` counts pages
2. **Find orphans fast:** Look for pages not in any `related:` field
3. **Track by tag:** Search for pages tagged #performance, #monorepo, etc. to check related topics
4. **Check last_updated:** `grep "last_updated:" wiki/**/*.md | sort` shows which pages are stale
5. **Use Obsidian graph:** Visualize wiki structure; orphans appear isolated
6. **Lint after major changes:** After ingesting 10+ pages or making significant updates

---

## Common Lint Issues & Fixes

| Issue | Symptom | Fix |
|-------|---------|-----|
| Stale content | Page references old Bazel version | Update content, update `last_updated` |
| Orphan pages | Page appears in index but not linked from anywhere | Add `related:` entries to 2–3 relevant pages |
| Broken links | Wikilink points to non-existent page | Either create the page or remove/update the link |
| Contradictions | Two pages claim opposite things | Note discrepancy in both pages or resolve to one |
| Missing cross-refs | Related pages don't link to each other | Add bidirectional `related:` entries |
| Incomplete sections | Page has TODO or [incomplete] markers | Complete or flag for follow-up source |
| Duplicate pages | Concept covered in two different pages | Consolidate into one, redirect others |
```

- [ ] **Step 2: Verify lint.md was created**

```bash
test -f .claude/skills/lint.md && echo "lint.md exists" || echo "Error: lint.md not found"
```

Expected: `lint.md exists`

- [ ] **Step 3: Commit**

```bash
git add .claude/skills/lint.md
git commit -m "docs: add lint skill for wiki health checks

Documents step-by-step workflow for linting: check contradictions, spot stale
claims, find orphans, identify gaps, verify cross-references, scan for
incomplete sections, log results. Includes verification checklist, cadence
guidelines, examples, and common issue fixes.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Task 8: Create wiki/index.md (Stub)

**Files:**
- Create: `wiki/index.md`

**Interfaces:**
- Produces: Initial index stub that will grow as pages are ingested

---

- [ ] **Step 1: Create index.md stub**

Create file `wiki/index.md`:

```markdown
# Bazel Wiki Index

This is the catalog of all pages in the Bazel LLM Wiki. Updated after every ingest.

**Status:** Just initialized; awaiting first ingest of official Bazel documentation.

---

## Concepts

### Fundamentals
(No pages yet. First ingest will add pages like targets, rules, artifacts, BUILD files, etc.)

### Advanced
(No pages yet. Will be populated after fundamentals are solid.)

---

## Reference
(No pages yet. Will contain synthesis of official Bazel docs: CLI reference, built-in rules, API guides, configuration.)

---

## Languages
(No pages yet. Will contain guides for Python, Java, C++, TypeScript, and other languages.)

---

## Tools
(No pages yet. Will contain guides for Gazelle, Buildifier, Buildozer, and other tools.)

---

## Patterns
(No pages yet. Will contain best practices: monorepo layout, testing strategy, dependency management, custom rules, etc.)

---

## Troubleshooting
(No pages yet. Will contain debugging guides, performance optimization, common issues, etc.)

---

## Experiments
(No pages yet. Will contain pages documenting your hands-on learning experiments and projects.)

---

**Next Step:** Ingest the official Bazel concepts guide to populate concepts/fundamentals/ and reference/.
```

- [ ] **Step 2: Verify index.md was created**

```bash
test -f wiki/index.md && echo "index.md exists" || echo "Error: index.md not found"
```

Expected: `index.md exists`

- [ ] **Step 3: Commit**

```bash
git add wiki/index.md
git commit -m "chore: create wiki/index.md stub

Initialize empty index organized by category. Will be populated by LLM
during ingest operations.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Task 9: Create wiki/log.md (Stub)

**Files:**
- Create: `wiki/log.md`

**Interfaces:**
- Produces: Initial log stub that will be appended to with every ingest, query, and lint operation

---

- [ ] **Step 1: Create log.md stub**

Create file `wiki/log.md`:

```markdown
# Bazel Wiki Activity Log

Append-only timeline of all ingest, query, and lint operations on the wiki.

Each entry starts with `## [YYYY-MM-DD] <operation> | <description>` so it's grep-able:
`grep "^## \[" log.md` shows all entries.

---

## Log Format

```
## [YYYY-MM-DD] <operation> | <description>

- Created: [[page1]], [[page2]]
- Updated: [[page3]]
- Key findings: Summary
- [Operation-specific fields]
```

Operations:
- **ingest** — Added new source to wiki
- **query** — Answered a question, possibly creating new page
- **lint** — Ran health check on wiki

---

## Activity

(Awaiting first operations...)

---

**Next:** First ingest will be official Bazel concepts guide.
```

- [ ] **Step 2: Verify log.md was created**

```bash
test -f wiki/log.md && echo "log.md exists" || echo "Error: log.md not found"
```

Expected: `log.md exists`

- [ ] **Step 3: Commit**

```bash
git add wiki/log.md
git commit -m "chore: create wiki/log.md stub

Initialize append-only activity log. Will record ingest, query, and lint
operations with parseable format (starts with \`## [YYYY-MM-DD]\`).

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Task 10: Create .gitignore

**Files:**
- Create: `.gitignore`

**Interfaces:**
- Produces: Git ignore rules for excluding raw sources and system files

---

- [ ] **Step 1: Create .gitignore**

Create file `.gitignore`:

```
# Raw sources — exclude large files and external content
raw/

# Obsidian cache and plugins
.obsidian/cache/
.obsidian/plugins/
.obsidian/*.json
!.obsidian/vault.json

# Claudian cache
.claudian/

# System files
.DS_Store
Thumbs.db
*.swp
*.swo
*~

# IDE
.vscode/
.idea/
*.sublime-project
*.sublime-workspace

# OS-specific
*~
.*.sw[a-z]

# Temporary files
*.tmp
*.bak
```

- [ ] **Step 2: Verify .gitignore was created**

```bash
test -f .gitignore && echo ".gitignore exists" || echo "Error: .gitignore not found"
```

Expected: `.gitignore exists`

- [ ] **Step 3: Commit**

```bash
git add .gitignore
git commit -m "chore: add .gitignore

Exclude raw sources (immutable external content), Obsidian cache/plugins,
Claudian cache, and OS/IDE-specific files from git tracking.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Task 11: Verify Complete Vault Structure & Summary

**Files:**
- Verify: All folders created
- Verify: All files created
- Verify: Git history

**Interfaces:**
- Produces: Confirmation that vault is ready for first ingest

---

- [ ] **Step 1: Verify complete folder structure**

Run from vault root:

```bash
find . -type d | grep -E "(raw|wiki|\.claude)" | sort
```

Expected output includes:
```
./.claude/skills
./raw/articles
./raw/assets
./raw/books
./raw/docs
./raw/experiments
./raw/videos
./wiki/concepts/advanced
./wiki/concepts/fundamentals
./wiki/experiments
./wiki/languages
./wiki/patterns
./wiki/reference
./wiki/tools
./wiki/troubleshooting
```

- [ ] **Step 2: Verify all key files exist**

```bash
echo "=== Checking .claude/ ===" && ls -la .claude/ && \
echo -e "\n=== Checking .claude/skills/ ===" && ls -la .claude/skills/ && \
echo -e "\n=== Checking wiki/ stubs ===" && ls -la wiki/index.md wiki/log.md
```

Expected: All files exist and are readable

- [ ] **Step 3: Verify .gitignore is working**

```bash
git status
```

Expected: `raw/` should not appear in status; no "untracked" raw files

- [ ] **Step 4: Review git log**

```bash
git log --oneline | head -10
```

Expected: Should see all commits we made:
- .gitignore
- wiki/log.md
- wiki/index.md
- .claude/skills/lint.md
- .claude/skills/query.md
- .claude/skills/ingest.md
- CLAUDE.md
- wiki/ structure
- raw/ structure
- Design spec

- [ ] **Step 5: Quick sanity check on CLAUDE.md**

```bash
head -50 .claude/CLAUDE.md | grep -E "(Three-Layer|Raw Sources|Wiki|Schema)" | wc -l
```

Expected: >= 4 (confirms sections exist)

- [ ] **Step 6: Verify frontmatter schema is clear**

```bash
grep -A 5 "^## Frontmatter Schema" .claude/CLAUDE.md | head -10
```

Expected: Shows schema with title, category, level, status fields

- [ ] **Step 7: Final status check**

```bash
echo "=== Vault Status ===" && \
echo "Raw folders: $(find raw -type d | wc -l)" && \
echo "Wiki folders: $(find wiki -type d | wc -l)" && \
echo "Total commits: $(git rev-list --all --count)" && \
echo "Current branch: $(git branch --show-current)"
```

Expected:
```
Raw folders: 7
Wiki folders: 10
Total commits: 11
Current branch: main
```

- [ ] **Step 8: Commit verification summary (optional, only if any changes)**

If no changes, skip. Otherwise:

```bash
git add -A && git commit -m "chore: vault initialization complete

Bazel LLM Wiki is initialized and ready for first ingest.
Structure: raw/ (sources), wiki/ (knowledge base), .claude/ (schema).
All workflows (ingest/query/lint) documented in skills.

Next: Ingest official Bazel documentation.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>"
```

---

## Summary

**Vault is now initialized with:**

✅ **Raw sources folder** — `raw/articles/`, `raw/docs/`, `raw/books/`, `raw/videos/`, `raw/experiments/`, `raw/assets/`

✅ **Wiki folder structure** — Organized by category and learning level:
- `wiki/concepts/fundamentals/` and `wiki/concepts/advanced/`
- `wiki/reference/`, `wiki/languages/`, `wiki/tools/`, `wiki/patterns/`, `wiki/troubleshooting/`, `wiki/experiments/`

✅ **Schema layer (`.claude/`):**
- `CLAUDE.md` — Master schema with principles, structure, conventions
- `.claude/skills/ingest.md` — Workflow for adding new sources
- `.claude/skills/query.md` — Workflow for asking questions
- `.claude/skills/lint.md` — Workflow for health checks

✅ **Index and log:**
- `wiki/index.md` — Catalog (stub, will be populated)
- `wiki/log.md` — Activity timeline (stub, append-only)

✅ **Git setup:**
- `.gitignore` — Excludes raw sources, caches, system files
- Clean git history with 11 focused commits

**Ready for first ingest!** See [[.claude/skills/ingest.md]] to process your first source (official Bazel documentation recommended).

