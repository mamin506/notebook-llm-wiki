# Bazel LLM Wiki Schema

**Vault Purpose:** Persistent, compounding knowledge base for Bazel learning (fundamentals through mastery)

**Philosophy:** Unlike RAG systems that re-derive knowledge on every query, this wiki *accumulates* knowledge. Cross-references are already in place, contradictions are flagged, synthesis reflects everything learned so far. The wiki compounds with every ingested source and every query explored.

**Design Pattern:** Follows the three-layer architecture outlined by Karpathy's LLM Wiki pattern:
- **Raw sources** (immutable) feed into the wiki
- **The wiki** (LLM-generated) synthesizes and cross-references knowledge
- **The schema** (this file + skills) governs how the LLM maintains it

---

## Design Goals

1. **Progressive Learning** — Support learning from fundamentals → intermediate → advanced
2. **Interactive & Iterative** — Add sources one at a time, explore with questions, build incrementally
3. **Comprehensive Scope** — Cover Bazel core, languages, tools, tooling, and integrations
4. **Maintainability** — LLM does the bookkeeping; you focus on curation and direction
5. **Traceability** — Every wiki page traces back to source(s); every ingest/query is logged
6. **Extensibility** — Easy to add new categories, workflows, or conventions as needs evolve

---

## Three-Layer Architecture

### Layer 1: Raw Sources (Immutable)

**Directory:** `raw/`

Immutable collection of source documents. The LLM reads from here but never modifies.

```
raw/
├── inbox/                 # Newly added sources waiting to be ingested
│   ├── articles/         # Incoming article clippings and notes
│   ├── books/            # Incoming book chapters or long-form content
│   ├── docs/             # Incoming documentation and reference material
│   ├── experiments/      # Incoming learning experiments or notes
│   └── videos/           # Incoming transcripts or video notes
├── processed/             # Sources already ingested and archived
│   ├── articles/
│   ├── books/
│   ├── docs/
│   ├── experiments/
│   └── videos/
└── assets/                # Images, diagrams, screenshots, and extracted media
```

**Principle:** Raw sources remain unchanged. The active ingest queue is `raw/inbox/`; once a source is processed, it should be moved into the matching subfolder under `raw/processed/`. This keeps pending-vs-archived state explicit without needing any LLM-maintained status list, while the source-type folders provide a simple, durable structure for the raw corpus.

### Layer 2: Wiki (LLM-Maintained Knowledge Base)

**Directory:** `wiki/`

LLM-generated and maintained knowledge base organized by category and learning level.

```
wiki/
├── concepts/          # Core Bazel ideas
│   ├── fundamentals/  # Learning level: beginner
│   └── advanced/      # Learning level: expert
├── reference/         # Synthesis of official documentation
├── languages/         # Language-specific guides (Python, Java, C++, TypeScript)
├── tools/             # Ecosystem tools (Buildifier, Gazelle, Buildozer)
├── patterns/          # Best practices, design patterns, architectural guidance
├── troubleshooting/   # Debugging, performance, common issues
├── experiments/       # Pages derived from YOUR hands-on work
├── index.md          # Catalog of all wiki pages (searchable index)
└── log.md            # Append-only activity log (ingest, query, lint records)
```

**Organization Principles:**
- *Category* determines folder (concepts, reference, languages, etc.)
- *Learning level* determines subfolder within concepts (fundamentals, advanced)
- *Status* tracked in frontmatter (seedling, growing, mature, complete)
- *Tags* enable cross-cutting themes (performance, monorepo, debugging)

### Layer 3: Schema Layer (This File + Skills)

**Files:** `.claude/CLAUDE.md`, `./claude/docs/superpowers/`, `.claude/skills/ingest/SKILL.md`, `.claude/skills/query/SKILL.md`, `.claude/skills/lint/SKILL.md`

Configuration and workflow instructions that tell the LLM:
- How the wiki is structured
- What conventions to follow
- How to ingest sources, answer queries, and maintain health
- Where to store superpower guidance documents: use `./claude/docs/superpowers/` for rule files and supporting documentation that should be treated as project-level superpower guidance rather than raw source material

---

## Folder Structure (Complete)

### concepts/ — Core Bazel Ideas

What are targets, rules, artifacts, BUILD files, dependency graphs, workspace configuration?

**Subfolder structure:**
- `fundamentals/` — Beginner-level concepts (what is a target, what is a rule)
- `advanced/` — Expert-level concepts (macros, aspect-driven development, custom rules)

**Example pages:**
- `concepts/fundamentals/targets.md` — What targets are, why they matter
- `concepts/fundamentals/rules.md` — What rules are, basic rule anatomy
- `concepts/advanced/macros.md` — DSL abstractions, when/why to use macros

### reference/ — Synthesis of Official Docs

Structured reference material synthesized from official Bazel documentation.

**Example pages:**
- `reference/cli-reference.md` — Bazel CLI commands, flags, options
- `reference/builtin-rules.md` — Catalog of built-in rules
- `reference/api-guide.md` — Bazel API reference
- `reference/configuration.md` — Workspace and build configuration

### languages/ — Language-Specific Guides

How to use Bazel with specific programming languages.

**Example pages:**
- `languages/python.md` — Python monorepos, py_library, py_binary, pytest integration
- `languages/java.md` — Java rules, maven/gradle interop, testing
- `languages/cpp.md` — C++ rules, external dependencies, cross-compilation
- `languages/typescript.md` — TypeScript/JavaScript, npm, bundling

### tools/ — Ecosystem Tooling

Tools that extend or augment Bazel workflows.

**Example pages:**
- `tools/gazelle.md` — Automatic BUILD file generation
- `tools/buildifier.md` — BUILD file formatting and linting
- `tools/buildozer.md` — BUILD file manipulation and refactoring
- `tools/bazel-run.md` — Running binaries and tests locally

### patterns/ — Best Practices & Patterns

Actionable guidance on common patterns and design decisions.

**Example pages:**
- `patterns/monorepo-layout.md` — Structuring a monorepo with Bazel
- `patterns/cross-platform-builds.md` — Supporting multiple platforms
- `patterns/dependency-management.md` — Handling external dependencies
- `patterns/test-strategy.md` — Testing philosophy and test pyramids
- `patterns/custom-rules.md` — When/how to write custom rules

### troubleshooting/ — Debugging & Common Issues

Solutions to problems, performance optimization, debugging techniques.

**Example pages:**
- `troubleshooting/build-failures.md` — Diagnosing and fixing build errors
- `troubleshooting/performance.md` — Build performance optimization
- `troubleshooting/cache-issues.md` — Remote cache, incremental builds
- `troubleshooting/dependency-hell.md` — Resolving circular/conflicting dependencies

### experiments/ — Your Hands-On Work

Pages documenting YOUR learning experiments, custom projects, and discoveries.

**Example pages:**
- `experiments/my-first-workspace.md` — Building your first Bazel workspace
- `experiments/custom-rule-exploration.md` — Exploring custom rule development
- `experiments/monorepo-migration.md` — Migrating an existing project to Bazel

---

## Frontmatter Schema & Metadata

Every wiki page opens with YAML frontmatter that tracks metadata and learning progression.

### Standard Frontmatter Template

```yaml
---
title: "Page Title"
category: "concepts|reference|languages|tools|patterns|troubleshooting|experiments"
level: "fundamentals|intermediate|advanced"
status: "seedling|growing|mature|complete"
sources: ["source-file-1.md", "source-file-2.md"]
tags: ["tag1", "tag2"]
related: ["[[other-page]]", "[[another-page]]"]
last_updated: "2026-07-17"
---
```

### Field Definitions

| Field | Purpose | Values | Notes |
|-------|---------|--------|-------|
| `title` | Page title | String | Clear, specific titles preferred |
| `category` | Wiki section | concepts, reference, languages, tools, patterns, troubleshooting, experiments | Determines folder placement |
| `level` | Learning progression | fundamentals, intermediate, advanced | "fundamentals" ≈ beginner, "intermediate" ≈ intermediate, "advanced" ≈ expert |
| `status` | Completion state | seedling, growing, mature, complete | *seedling*: minimal content from one source; *growing*: multiple sources, expanding; *mature*: comprehensive; *complete*: unlikely to change |
| `sources` | Traceability | List of raw file names | Links back to raw/ directory; enables impact analysis |
| `tags` | Cross-cutting themes | List of tags (e.g., #monorepo, #performance, #debugging) | Enables queries across categories |
| `related` | Cross-references | Wikilinks to related pages | LLM maintains these; the wiki's nervous system |
| `last_updated` | Freshness tracking | ISO date (YYYY-MM-DD) | Helps spot stale content during lint passes |

### Page Types & Conventions

| Type | Typical Length | Purpose | Example |
|------|---|---|---|
| **Concept** | 200–800 words | Explain one idea deeply, with context and examples | "Targets: The Atomic Unit of Bazel" |
| **Synthesis** | 800–2000 words | Integrate multiple sources, show connections, resolve conflicts | "Macros vs Rules: When to Use Each" |
| **Reference** | 500–1500 words | Structured reference (commands, flags, API methods) | "Bazel CLI Reference" |
| **Pattern/Guide** | 1000–2000 words | Step-by-step practical advice with examples | "Setting Up a Python Monorepo with Bazel" |
| **Experiment Log** | 300–1000 words | Document YOUR hands-on work, questions, findings, lessons | "Custom Java Rules: Exploration & Design" |

**Creation Principles:**
- Favor *specificity* over comprehensiveness (one concept per page)
- Use Bazel terminology consistently
- When integrating contradictory sources, note the discrepancy
- Link early and often (cross-references are the wiki's strength)

---

## Workflows

### INGEST: Adding a New Source

**Trigger:** You add a new file to `raw/inbox/` or ask the LLM to process pending sources.

**Workflow Steps:**

1. **Scan the Active Queue**
   - Check `raw/inbox/` for pending sources
   - If the inbox is empty, do nothing and stop
   - Treat every file currently in `raw/inbox/` as new and unprocessed

2. **Read & Analyze**
   - Read each source carefully
   - Identify key concepts, principles, examples, contradictions with existing wiki
   - Note gaps (things mentioned but not explained)

3. **Create/Update Pages**
   - For each concept found:
     - Does a wiki page exist? If yes, read it
     - Integrate new information: update content, add examples, flag contradictions
     - If no page exists, create one with appropriate category/level
   - Update frontmatter: `status`, `sources`, `last_updated`, `related`

4. **Update Cross-References**
   - Review pages that *should* link to the new content
   - Add wikilinks where relevant
   - Update `related` fields in both directions (if A links to B, B should link to A)

5. **Update Index**
   - Add new pages to `wiki/index.md`
   - Update entry summaries if existing pages changed substantially
   - Keep index organized by category

6. **Archive the Processed Sources**
   - After successful ingest, move each processed file from `raw/inbox/` to `raw/processed/`
   - Use a shell command such as `mv raw/inbox/* raw/processed/` or move them manually in the file explorer
   - If a file needs to be reprocessed, move it back from `raw/processed/` to `raw/inbox/`

7. **Log the Ingest**
   - Append entry to `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] ingest | [Source Title]
   
   Created: [[page1]], [[page2]]
   Updated: [[page3]], [[page4]]
   Key findings: [Summary of main insights]
   Open questions: [Questions raised but not yet answered]
   ```

**Ingest Goals:**
- Extract *actionable* knowledge, not just summaries
- Connect new content to existing wiki (avoid isolated pages)
- Ask clarifying questions if ambiguous
- Suggest follow-up sources based on gaps found
- Maintain Bazel terminology consistency

**Expected Output:**
- 1–10 new or updated wiki pages
- Index updated
- Log entry appended
- One discussion (clarifying questions, suggestions)

---

### QUERY: Asking Questions

**Trigger:** You ask a question about Bazel (or about the wiki itself).

**Workflow Steps:**

1. **Search the Index**
   - Read `wiki/index.md` to find relevant pages
   - Look for pages matching the query topic

2. **Read Relevant Pages**
   - Drill into pages identified above
   - Note cross-references and related ideas
   - Read linked pages if they add context

3. **Synthesize & Answer**
   - Combine information from pages with citations
   - Call out contradictions or uncertainty
   - Answer in whatever format fits (markdown, table, comparison, etc.)

4. **File if Valuable**
   - If your answer reveals a new synthesis, pattern, or useful comparison:
     - Create a new wiki page (e.g., comparison of two approaches)
     - Update index and cross-references
   - Not every query result needs to be filed—only valuable, reusable insights

5. **Log the Query**
   - Append entry to `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] query | [Question Summary]
   
   Pages referenced: [[page1]], [[page2]]
   Answer: [Summary]
   New page created: [[new-synthesis-page]] (if applicable)
   Gaps identified: [Notable missing info or contradictions]
   ```

**Query Goals:**
- Use the wiki as-is (don't ignore structure)
- Surface contradictions and gaps
- Suggest sources to fill gaps
- File valuable results back into the wiki (learning by exploring compounds)

**Expected Output:**
- Answer to your question
- Possibly a new wiki page (synthesis, comparison, or guide)
- Log entry
- Possibly suggestions for follow-up sources or experiments

---

### LINT: Periodic Health Checks

**Trigger:** Monthly, or after major ingest batches. You can request a lint pass anytime.

**Workflow Steps:**

1. **Check for Contradictions**
   - Find pages claiming opposite things
   - Flag which sources caused the conflict
   - Resolve: update one page, note discrepancy if both are valid, suggest new source

2. **Spot Stale Claims**
   - Look for facts that newer sources have superseded
   - Check `last_updated` dates (old pages may be stale)
   - Update or mark for verification

3. **Find Orphan Pages**
   - Pages with no inbound links (except the index)
   - Either: integrate into a parent page, or add cross-references, or delete if truly obsolete

4. **Identify Gaps**
   - Important concepts mentioned in multiple pages but lacking their own page
   - Create new pages or expand existing ones

5. **Check Cross-References**
   - Find pages that *should* be linked but aren't
   - Add missing `related` entries

6. **Scan for Incomplete Sections**
   - Pages marked "TODO" or with incomplete explanations
   - Either complete, or flag for follow-up sources

7. **Log the Lint Pass**
   - Append entry to `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] lint | Monthly health check
   
   Contradictions found: 2 (both resolved)
   Stale claims: 3 (updated with newer sources)
   Orphan pages: 4 (3 linked, 1 deleted)
   Gaps: 5 (1 new page created, 4 marked for follow-up)
   Missing cross-refs: 8 (all fixed)
   Summary: Wiki is healthy, suggest exploring [gap X] with a new source
   ```

**Lint Goals:**
- Keep the wiki consistent and current
- Surface gaps and suggest follow-up sources
- Maintain the web of cross-references
- Generate questions worth exploring

**Expected Output:**
- Health report (contradictions, gaps, orphans)
- Fixed issues (pages updated, links added)
- New pages if gaps identified
- Log entry
- Suggested follow-up sources

---

## Frontmatter Discipline & Maintenance

### Keeping Status Current

The `status` field drives awareness of wiki maturity:

- **seedling** — Just created from a source, minimal content, likely incomplete
- **growing** — Multiple sources integrated, expanding with examples, mostly coherent
- **mature** — Well-developed, comprehensive, cross-references in place, ready for reference
- **complete** — Unlikely to change much; expert-level coverage

**Progression Example:**
```
2026-07-17: [[concepts/fundamentals/targets]] created as "seedling" from official guide
2026-07-22: Ingest blog post, add examples, update to "growing"
2026-08-05: Ingest video transcript, refine explanation, update to "mature"
2026-09-01: Lint pass confirms mature, could mark "complete" if confident
```

### Updating last_updated

- Update `last_updated` *every* time you edit a page
- Enables lint to find stale content: pages with `last_updated` > 3 months old may need review
- Format: ISO date (YYYY-MM-DD)

### Maintaining Related Links

The `related` field is *critical*—it's how the wiki becomes a web of knowledge, not a pile of pages.

**Guidelines:**
- When you create a new page, immediately identify related pages and add wikilinks
- When you edit a page, check if new relationships emerged
- Bidirectional: if A links to B, B should link to A
- Update `related` during ingest/query/lint workflows

### Tags for Cross-Cutting Themes

Use tags to mark pages that belong to themes that span categories.

**Example tags:**
- `#monorepo` — Pages about monorepo architecture (concepts, patterns, languages)
- `#performance` — Build performance, caching, optimization (troubleshooting, patterns)
- `#testing` — Testing strategy, test rules, test languages (patterns, languages)
- `#debugging` — Troubleshooting, debugging techniques (troubleshooting)
- `#external-deps` — Managing external dependencies (patterns, troubleshooting)

**Query with tags:** At query time, the LLM can search by tag to find related pages across categories.

### Source Traceability

Always track which raw files contributed to a page:

```yaml
sources: ["docs/concepts-guide.md", "blog-post-macros.md", "your-experiment-abc.md"]
```

**Enables:**
- Impact analysis ("if this source is wrong, which pages are affected?")
- Prioritization ("which pages need updates based on newest sources?")
- Credibility ("what's the evidence for this claim?")

---

## Index and Log Conventions

### wiki/index.md — The Wiki Catalog

**Purpose:** Searchable, up-to-date catalog of all wiki pages. Helps you and the LLM navigate.

**Format:**
```markdown
# Bazel Wiki Index

## Concepts

### Fundamentals
- [[concepts/fundamentals/targets]] — The atomic unit of Bazel builds (seedling)
- [[concepts/fundamentals/rules]] — Reusable build instructions (growing)
- [[concepts/fundamentals/artifacts]] — Outputs of rules; how Bazel tracks them (mature)

### Advanced
- [[concepts/advanced/macros]] — DSL abstractions for rule reuse (seedling)
- [[concepts/advanced/custom-rules]] — Writing your own rules (growing)

## Reference
- [[reference/cli-reference]] — Bazel CLI commands and flags (mature)
- [[reference/builtin-rules]] — Built-in rule catalog (mature)
- [[reference/api-guide]] — Bazel API (intermediate)

## Languages
- [[languages/python]] — Python in Bazel (growing)
- [[languages/java]] — Java in Bazel (seedling)

## Tools
- [[tools/gazelle]] — Automatic BUILD generator (seedling)
- [[tools/buildifier]] — BUILD file formatter (seedling)

## Patterns
- [[patterns/monorepo-layout]] — Structuring monorepos (seedling)

## Troubleshooting
- [[troubleshooting/build-failures]] — Debugging build errors (seedling)

## Experiments
- [[experiments/my-first-workspace]] — Your first Bazel project (mature)
```

**Maintenance:**
- Update after every ingest (add new pages, update status)
- Organized by category, then by level (fundamentals → advanced where applicable)
- Format is parseable: `grep "^- " index.md` lists all pages; `grep "status\|growing" index.md` finds growing pages

### wiki/log.md — Activity Timeline

**Purpose:** Append-only record of ingests, queries, and lint passes. Shows wiki evolution over time.

**Format:**
```markdown
# Bazel Wiki Activity Log

## [2026-07-17] ingest | Official Bazel Concepts Guide

- Created: [[concepts/fundamentals/targets]], [[concepts/fundamentals/rules]], [[concepts/fundamentals/artifacts]]
- Updated: [[reference/api-guide]]
- Key insights: Targets are immutable atomic units; rules are parameterized build instructions
- Open questions: How do we express complex dependencies? (See [[concepts/fundamentals/dependencies]] for partial answer)
- Next source: Find a guide on dependency graphs

## [2026-07-18] query | How do macros differ from rules?

- Pages referenced: [[concepts/fundamentals/rules]], [[reference/api-guide]]
- Answer: Rules are executed by Bazel; macros are DSL abstractions that expand at load time
- New page: [[concepts/advanced/macros-vs-rules]] (synthesis)
- Gaps: Need practical guide on when to use each

## [2026-07-20] lint | Monthly health check

- Contradictions: 2 found (both resolved; noted in [[reference/api-guide]])
- Stale claims: 1 ([[languages/python]] flagged for update)
- Orphans: 3 (all linked to parent pages)
- Gaps: 5 important concepts lacking pages
- Missing cross-refs: 4 (all added)
- Recommendation: Add sources on custom rules, Python monorepos, performance optimization
```

**Format Convention:**
- Each entry starts with `## [YYYY-MM-DD]` so it's grep-able: `grep "^## \[" log.md`
- Entries are append-only (never modified after logging)
- Entries include: what happened, pages affected, key insights, open questions, recommendations

---

## Usage Tips

### For Working with the Wiki

1. **Start with the Index**
   - Before searching for anything, check `wiki/index.md`
   - The index is the compass that keeps you oriented

2. **Follow Cross-References**
   - Don't read pages in isolation; follow `related` links
   - The web of links *is* the knowledge

3. **Trust the Frontmatter**
   - Page `status` tells you maturity
   - `last_updated` tells you freshness
   - `sources` tells you evidence
   - `tags` help you find related pages across categories

4. **Use the Log for Context**
   - Read `wiki/log.md` to understand what's been explored and what gaps remain
   - Avoid duplicating work; check the log before asking a question

5. **When You Add Content**
   - Always start with frontmatter (title, category, level, status, sources, tags, related, last_updated)
   - Think about how your page relates to existing pages
   - Link early; link often
   - Update the index and log

6. **For Lint Passes**
   - Run monthly or after major ingests
   - Use the LINT skill for detailed guidance
   - Focus on contradictions, stale claims, orphans, and gaps

7. **Document Your Learning**
   - Create experiment/ pages for your hands-on work
   - Tag pages with cross-cutting themes (#monorepo, #performance, #testing, #debugging)
   - Every ingest/query/lint should update the log

---

## Related Resources

- **Design Spec:** `docs/superpowers/specs/2026-07-17-bazel-llm-wiki-design.md`
- **Ingest Skill:** `.claude/skills/ingest/SKILL.md`
- **Query Skill:** `.claude/skills/query/SKILL.md`
- **Lint Skill:** `.claude/skills/lint/SKILL.md`

---

**Last updated:** 2026-07-17  
**Status:** Active  
**Version:** 1.0
