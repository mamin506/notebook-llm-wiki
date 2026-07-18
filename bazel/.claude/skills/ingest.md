# Ingest Skill: Processing & Integrating New Sources

## Header

**Purpose:** Systematically read, analyze, and integrate new sources into the Bazel wiki. Each ingest extracts actionable knowledge, creates or updates wiki pages, maintains cross-references, and logs activity.

**When to Use:**
- You add a new file to `raw/` (docs, articles, blog posts, videos, experiments)
- You ask the LLM to "ingest [source]" or "process [document]"
- You want to expand the wiki with new material

**Expected Time:** 30–90 minutes depending on source length and complexity (10–40 pages typically takes 30–45 minutes; longer or more complex sources may take 60–90 minutes)

**Skill Level:** Intermediate (requires understanding wiki structure, frontmatter, and cross-referencing discipline)

---

## Workflow

### Step 1: Read & Analyze

**Goal:** Extract concepts, examples, contradictions, and gaps from the source.

**Actions:**

1. **Read the source carefully**
   - Take notes on key concepts, principles, and examples
   - Identify the source's intended audience and scope
   - Note any claims that seem controversial or need verification

2. **Identify key concepts**
   - What are the *core ideas* the source explains? (e.g., "What is a target?", "How do macros work?")
   - List them as separate bullet points
   - Estimate which wiki category each concept belongs to (concepts, reference, patterns, etc.)

3. **Check for contradictions**
   - Do any claims contradict what's already in the wiki?
   - Flag these with their source (e.g., "Source X says Y, but wiki page Z says otherwise")
   - Note if both may be valid (different contexts, versions, or interpretations)

4. **Identify gaps**
   - What does the source *mention* but not fully explain? (e.g., "macros are defined with a macro() function" but no explanation of how to write one)
   - What related topics are *not* covered? (e.g., a concepts guide that covers targets but skips the dependency graph)
   - These gaps become candidates for follow-up sources

5. **Note examples**
   - What concrete examples, code snippets, or walkthroughs does the source provide?
   - Which wiki pages would benefit from these examples?

**Output of Step 1:**
- Mental (or written) inventory of concepts and their categories
- List of contradictions to resolve
- List of gaps to address
- Examples to integrate

---

### Step 2: Create/Update Wiki Pages

**Goal:** For each concept, create a new wiki page or integrate into an existing one.

**Actions:**

1. **For each concept identified in Step 1:**

   a. **Check if a page exists**
      - Search `wiki/index.md` for the concept
      - Search the wiki folder structure directly if needed
      - Example: If the source explains "Aspects", check `wiki/concepts/advanced/aspects.md`

   b. **If a page exists:**
      - Read it carefully to understand current coverage
      - Integrate new information:
        - Add new examples or clarifications
        - Expand explanations if the source provides more depth
        - Flag contradictions in a "Note" or "Contradiction" section (e.g., "Source X claims Y, but this may depend on Bazel version Z")
      - Update frontmatter:
        - Add source filename to `sources` list (if not already there)
        - Update `last_updated` to today's date
        - Review and update `status` (seedling → growing → mature based on coverage)
        - Add any new relevant tags
        - Update `related` if new connections emerged

   c. **If no page exists:**
      - Create a new page with appropriate category and level
      - Use the design spec's page templates as a guide
      - Write clear, focused content (avoid trying to explain everything; be specific)
      - Set frontmatter:
        - `title`: Clear, specific title
        - `category`: concepts, reference, languages, tools, patterns, troubleshooting, or experiments
        - `level`: fundamentals, intermediate, or advanced (see spec for guidance)
        - `status`: seedling (new from single source)
        - `sources`: [filename]
        - `tags`: Relevant cross-cutting tags
        - `related`: Links to related pages (populated in Step 3)
        - `last_updated`: Today's date
      - Place file in correct folder structure (e.g., `wiki/concepts/fundamentals/targets.md`)

2. **Maintain Bazel terminology consistency**
   - Use official Bazel terms (e.g., "targets" not "build targets", "rules" not "rule definitions")
   - If the source uses non-standard terminology, note the mapping (e.g., "Source calls them 'build instructions' but the wiki calls them 'rules'")

3. **Keep writing clear and accessible**
   - Favor depth over breadth (explain one concept well rather than skim many)
   - Use examples from the source when relevant
   - Link to foundational concepts readers should understand first

**Output of Step 2:**
- 1–10 new or updated wiki pages
- Updated frontmatter across all modified pages
- Clear, traceable integration of source material

---

### Step 3: Update Cross-References

**Goal:** Weave the new pages into the wiki's network of connections.

**Actions:**

1. **Identify related pages**
   - Which existing wiki pages should link to the new content?
   - Which new pages should reference existing pages?
   - Example: If you just created `concepts/fundamentals/macros.md`, it should link to `concepts/fundamentals/rules.md` and vice versa

2. **Add wikilinks**
   - Use the format `[[path/to/page]]` (e.g., `[[concepts/fundamentals/rules]]`)
   - Add these links to:
     - The body of pages (where contextually relevant)
     - The `related` field in frontmatter (cross-cutting connections)

3. **Update `related` fields bidirectionally**
   - If page A links to page B, page B should link back to page A
   - Use frontmatter `related` field for these structural connections
   - Example:
     ```yaml
     # In concepts/fundamentals/targets.md
     related: ["[[concepts/fundamentals/rules]]", "[[concepts/fundamentals/artifacts]]"]
     
     # In concepts/fundamentals/rules.md
     related: ["[[concepts/fundamentals/targets]]", "[[concepts/fundamentals/artifacts]]"]
     ```

4. **Review cross-category links**
   - Does a new concepts page relate to patterns? (e.g., "Custom Rules" → "Custom Rules Guide")
   - Does it relate to languages? (e.g., "Python Imports" → "Python Language Guide")
   - Does it relate to troubleshooting? (e.g., "Dependency Resolution" → "Dependency Hell")
   - Add these connections to `related` fields

**Output of Step 3:**
- Updated `related` fields in all affected pages
- Bidirectional wikilinks in place
- New pages integrated into the wiki's knowledge graph

---

### Step 4: Update Index

**Goal:** Keep `wiki/index.md` current so you and the LLM can navigate the wiki.

**Actions:**

1. **Open `wiki/index.md`**
   - Review its structure (organized by category)

2. **Add new pages to the index**
   - Insert new pages in the appropriate category section
   - Use the format from the spec:
     ```markdown
     - [[path/to/page]] — Brief description (status)
     ```
   - Example: `- [[concepts/fundamentals/macros]] — DSL abstractions for rule reuse (seedling)`

3. **Update existing entries if needed**
   - If you significantly expanded a page, update its summary
   - If you changed a page's status, update it in the index
   - Example: `(seedling)` → `(growing)` if you added multiple examples

4. **Maintain organization**
   - Within each category, keep entries sorted logically (e.g., fundamentals before advanced)
   - Alphabetical is fine if no other order makes sense

**Output of Step 4:**
- `wiki/index.md` updated with all new pages
- Existing entries updated if their status or summary changed
- Index remains a current catalog of the wiki

---

### Step 5: Log the Ingest

**Goal:** Record what happened, what you learned, and what's next.

**Actions:**

1. **Open `wiki/log.md`**
   - This is an append-only log of all ingest, query, and lint activities

2. **Append a new entry at the end**
   - Use this format:
     ```markdown
     ## [YYYY-MM-DD] ingest | [Source Title]
     
     Created: [[page1]], [[page2]], [[page3]]
     Updated: [[page4]], [[page5]]
     Key findings: [2-3 sentence summary of main insights]
     Open questions: [Questions raised but not yet answered]
     Contradictions: [Any contradictions found and how resolved]
     Next steps: [Suggested follow-up sources or areas to explore]
     ```

3. **Be specific**
   - List actual page paths, not just categories
   - Summarize *actionable* insights, not just facts
   - Call out open questions and contradictions (these guide future work)
   - Suggest specific follow-up sources based on gaps found

4. **Format consistently**
   - Start with `## [YYYY-MM-DD] ingest | ` so entries are grep-able
   - Use wikilinks `[[]]` for page references
   - Keep it concise (3–10 sentences typically)

**Example Log Entry:**
```markdown
## [2026-07-25] ingest | Official Bazel Concepts Guide

Created: [[concepts/fundamentals/targets]], [[concepts/fundamentals/rules]], [[concepts/fundamentals/artifacts]]
Updated: [[reference/api-guide]] (added examples)
Key findings: Targets are immutable specifications; rules are parameterized build templates. The dependency graph is foundational but not fully explained.
Open questions: How do Bazel resolve circular dependencies? How do visibility constraints work across large monorepos?
Contradictions: None found; source aligns with current wiki.
Next steps: Find source on dependency graphs, advanced visibility patterns, and circular dependency handling. Consider experiments with custom rule development.
```

**Output of Step 5:**
- New entry appended to `wiki/log.md`
- Clear record of what was ingested, learned, and still missing
- Guidance for future ingests

---

## Verification Checklist

Before considering an ingest complete, verify the following 12 items:

- [ ] **All key concepts identified and assigned to categories** — Every concept from the source has been placed in concepts/, reference/, patterns/, etc., with appropriate level (fundamentals/intermediate/advanced)

- [ ] **New pages created with complete frontmatter** — All new pages have title, category, level, status, sources, tags, related, and last_updated fields populated

- [ ] **Existing pages updated with new source in `sources` list** — Any page modified includes the source filename in its frontmatter `sources` field

- [ ] **Status fields reflect content maturity** — Pages with only basic content are marked `seedling`; pages with multiple sources and examples are marked `growing` or `mature`

- [ ] **Cross-references added bidirectionally** — If page A links to page B, page B has a reciprocal link back to page A in its `related` field

- [ ] **Contradictions documented** — Any contradictions with existing wiki content are flagged with sources and explanations (or resolved with updates)

- [ ] **Gaps identified for follow-up** — Open questions and missing concepts are noted (either in page "TODO" sections or in the log)

- [ ] **Bazel terminology consistent** — Pages use official Bazel terms (targets, rules, aspects, macros, etc.) consistently; non-standard terminology is mapped

- [ ] **Index updated with all new pages** — `wiki/index.md` includes all new pages with appropriate category, description, and status

- [ ] **Log entry appended to wiki/log.md** — A new entry with date, source title, created/updated pages, key findings, and open questions is added

- [ ] **Wikilinks are valid** — All `[[path/to/page]]` references point to actual wiki pages (using the format `[[category/level/page]]` or `[[category/page]]` without .md extension)

- [ ] **No orphan pages** — No new pages are left without any inbound links (except the index)

---

## Bazel-Specific Tips

When ingesting sources about Bazel, keep these domain-specific considerations in mind:

### 1. **Distinguish Between Bazel Versions**
Bazel evolves. A feature in Bazel 6.0 might not exist in Bazel 4.0, or behavior may differ.
- **Action:** If the source mentions a Bazel version, note it in the page (e.g., "Available in Bazel 5.0+")
- **Flag contradictions:** If different sources claim different behaviors for the same feature, check if they're describing different versions
- **Example page note:** "Aspect implementation changed in Bazel 5.0; see [[reference/bazel-version-history]] for details"

### 2. **Recognize Rule Types & Rule Sets**
Bazel has built-in rules (`py_library`, `java_binary`) and external rule sets (`rules_python`, `rules_java`).
- **Action:** When documenting a rule, clarify which rule set it belongs to
- **Flag confusion:** Don't conflate a language (Python) with its rule set (rules_python) or individual rules (py_library)
- **Cross-reference:** Link language-specific pages to their rule sets (e.g., `languages/python.md` should reference `tools/rules-python.md` or similar)

### 3. **Label Syntax Matters**
Bazel labels have specific syntax: `//path/to/package:target`, `@external_repo//path:target`, `:local`, etc.
- **Action:** When explaining labels, be precise about syntax. Show examples
- **Document variants:** Explain absolute labels (`//`), package-relative (`:target`), external repo labels (`@`), etc.
- **Link to reference:** Pages explaining labels should link to `[[reference/label-syntax]]` or similar

### 4. **Monorepo Patterns Are Essential**
Most Bazel use cases involve monorepos. Ingested sources often touch on monorepo layout, dependency management, or scaling.
- **Action:** When you encounter monorepo-related content, explicitly tag it (`#monorepo`) and link it to `[[patterns/monorepo-layout]]`
- **Identify gaps:** After ingesting a source, ask: "Are there monorepo considerations not covered here?" (e.g., buildifier, dependency organization, cross-team visibility)
- **Cross-reference:** Monorepo content in languages/ should link to patterns/monorepo-layout.md

### 5. **Build Language vs. Starlark vs. Python**
Confusion often arises around the BUILD file language. The language is Starlark, and Starlark is similar to Python but not Python.
- **Action:** When a source discusses BUILD files or macros, clarify: "This uses Starlark, a Python-like language"
- **Flag misunderstandings:** If a source conflates Starlark with Python (e.g., "Python on line 5"), note this and clarify
- **Link to reference:** Pages about build language should reference `[[reference/starlark-syntax]]` or `[[reference/build-language]]`

---

## Example: Ingesting the Official Concepts Guide

**Source File:** `raw/docs/official-concepts-guide.md` (hypothetical)

**Step 1: Read & Analyze**
- The guide covers: targets, rules, labels, artifacts, the workspace, BUILD files, and the dependency graph
- Key concepts: 6 major concepts identified
- Contradictions: None (it's official documentation)
- Gaps: The guide mentions "external dependencies" but doesn't explain how to manage them; it doesn't cover advanced topics like aspects or macros
- Examples: Plenty of code snippets showing label syntax, simple rule definitions

**Step 2: Create/Update Pages**
- **New pages created:**
  - `wiki/concepts/fundamentals/targets.md` (seedling, from this source)
  - `wiki/concepts/fundamentals/rules.md` (seedling, from this source)
  - `wiki/concepts/fundamentals/labels.md` (seedling, from this source)
  - `wiki/concepts/fundamentals/artifacts.md` (seedling, from this source)
  - `wiki/concepts/fundamentals/workspace.md` (seedling, from this source)
  - `wiki/concepts/fundamentals/build-files.md` (seedling, from this source)

- **Updated pages:**
  - `wiki/reference/api-guide.md` (added "Label Reference" section with examples from source)

**Step 3: Update Cross-References**
- All six new pages link to each other in `related`:
  - `targets.md` → targets relate to rules, artifacts, and labels
  - `rules.md` → rules relate to targets and artifacts
  - `artifacts.md` → artifacts are outputs of rules and targets
  - `labels.md` → labels name targets; used in rule dependencies
  - `workspace.md` → workspace contains BUILD files and defines the scope of labels
  - `build-files.md` → BUILD files contain targets and rules

**Step 4: Update Index**
```markdown
## Concepts

### Fundamentals
- [[concepts/fundamentals/targets]] — The atomic unit of Bazel builds (seedling)
- [[concepts/fundamentals/rules]] — Reusable build instructions (seedling)
- [[concepts/fundamentals/labels]] — Unique names for targets (seedling)
- [[concepts/fundamentals/artifacts]] — Outputs of rules and how Bazel tracks them (seedling)
- [[concepts/fundamentals/workspace]] — Your Bazel project root and configuration (seedling)
- [[concepts/fundamentals/build-files]] — BUILD files define targets (seedling)

## Reference
- [[reference/api-guide]] — Bazel API, commands, and label syntax (intermediate)
```

**Step 5: Log the Ingest**
```markdown
## [2026-07-25] ingest | Official Bazel Concepts Guide

Created: [[concepts/fundamentals/targets]], [[concepts/fundamentals/rules]], [[concepts/fundamentals/labels]], [[concepts/fundamentals/artifacts]], [[concepts/fundamentals/workspace]], [[concepts/fundamentals/build-files]]
Updated: [[reference/api-guide]] (added label reference section)
Key findings: Targets are immutable specifications of build work; rules are parameterized templates. Labels follow `//path/to/package:target_name` syntax. The dependency graph is foundational but not fully explained in this source.
Open questions: How does Bazel resolve circular dependencies? How do advanced topics like aspects and macros fit? What about external dependencies and workspace integration?
Contradictions: None; this is official documentation and aligns with our wiki principles.
Next steps: Ingest a guide on dependency graphs and external dependencies. Find sources on advanced topics (macros, aspects). Suggest hands-on experiment: create a first Bazel workspace using these concepts.
```

**Expected Outcomes:**
- 6 new seedling pages in `wiki/concepts/fundamentals/`
- All 6 pages cross-linked
- 1 existing page updated (`reference/api-guide.md`)
- Index updated with 6 entries
- Log entry appended
- Clear foundation for future ingests on dependencies, advanced topics, and domain-specific content

---

## Common Pitfalls & How to Avoid Them

### 1. **Creating Isolated Pages (No Cross-References)**
**Pitfall:** You create a new page but don't link it to related content. The page becomes orphaned, invisible in the wiki's knowledge graph.
- **Example:** You ingest a guide on custom rules and create `wiki/concepts/advanced/custom-rules.md`, but you forget to link it to `concepts/fundamentals/rules.md` or `patterns/custom-rules.md`.
- **How to Avoid:** After creating any page, immediately identify 2–3 related pages. Add them to the `related` field. Check those pages too and add reciprocal links.
- **Verification:** Use the checklist item "No orphan pages" — search for pages with no inbound links (except the index).

### 2. **Mixing Contradictions Into the Same Page Without Noting Them**
**Pitfall:** Two sources say different things, so you blend them into one page without flagging the discrepancy. Future readers won't know which claim to trust.
- **Example:** Source A says "Aspects are applied during the analysis phase", Source B says "Aspects are applied during the action phase". You merge them without noting the contradiction.
- **How to Avoid:** When you find a contradiction:
  1. Create a "Note" or "Contradiction" section in the page
  2. State both claims with their sources: "Source A claims X; Source B claims Y"
  3. Offer a resolution: different contexts, versions, or a request for clarification
  4. Log the contradiction in wiki/log.md
- **Example Contradiction Note:**
  ```markdown
  **Note on Aspect Timing:** Source A claims aspects apply during analysis; Source B claims action phase.
  Both are correct: core aspect computation (gathering dependencies) happens during analysis;
  aspect actions execute during the build phase. See [[reference/aspects-detailed]] for clarification.
  ```

### 3. **Forgetting to Update Frontmatter**
**Pitfall:** You update a page's content but don't update `last_updated`, `sources`, or `status`. Months later, lint finds stale content but the frontmatter doesn't explain why.
- **Example:** You ingest a new source and add examples to `wiki/patterns/monorepo-layout.md`, but you forget to add the new source to the `sources` list or update `last_updated`.
- **How to Avoid:** After every edit, update:
  - `sources`: Add new source filename if you drew information from it
  - `last_updated`: Set to today's date
  - `status`: Bump if coverage improved (seedling → growing, growing → mature)
  - `related`: Add new connections if they emerged
- **Checklist Verification:** "Existing pages updated with new source in `sources` list" and "Status fields reflect content maturity"

### 4. **Ignoring Gaps Without Logging Them**
**Pitfall:** You notice gaps in the source (concepts mentioned but not explained, missing topics), but you don't record them. Later, you forget what follow-up sources to pursue.
- **Example:** You ingest a guide on custom rules, but it doesn't explain how rules interact with aspects. You note this mentally but don't log it.
- **How to Avoid:** In Step 1 (Read & Analyze), *explicitly* list gaps. In Step 5 (Log the Ingest), include a "Next steps" section that names specific follow-up sources or topics to explore.
- **Checklist Verification:** "Gaps identified for follow-up"

### 5. **Using Non-Standard Terminology Without Mapping**
**Pitfall:** A source uses non-standard or colloquial terms (e.g., "build target" instead of "target", "action" instead of "rule"), and you copy them into the wiki without clarification. The wiki's terminology becomes inconsistent.
- **Example:** A blog post uses "action" when it means "rule". You copy this into a wiki page without noting the terminology difference.
- **How to Avoid:**
  1. In Step 2 (Create/Update Pages), consciously use *official* Bazel terminology
  2. If the source uses non-standard terms, add a note: "This source uses 'action' to mean 'rule'; the wiki uses the official term 'rule'"
  3. Maintain a mental mapping as you work
- **Checklist Verification:** "Bazel terminology consistent"

### 6. **Creating Pages That Are Too Broad or Too Shallow**
**Pitfall:** You create a page that tries to cover too much (e.g., "All About Bazel Rules" in 500 words) or too little (e.g., "Aspects" in 100 words with no examples).
- **Example:** You create `wiki/concepts/fundamentals/everything.md` that attempts to explain targets, rules, artifacts, and labels all in one page. The page is confusing and not actionable.
- **How to Avoid:**
  - Follow the page template in the design spec: 200–800 words for concept pages, 800–2000 for synthesis
  - Create *one specific concept per page* (not "Everything About X")
  - If you need to cover multiple related concepts, create multiple pages and link them
  - Use the level field to signal complexity: fundamentals for beginners, advanced for experts
- **Checklist Verification:** "All key concepts identified and assigned to categories" — each concept gets its own page (or integrates into an existing focused page)

---

## Ingest Workflow Checklist (Quick Reference)

Use this as a quick reference while ingesting:

- [ ] **Step 1: Read & Analyze** — Identify concepts, contradictions, gaps
- [ ] **Step 2: Create/Update Pages** — Create new pages or integrate into existing; update frontmatter
- [ ] **Step 3: Update Cross-References** — Add bidirectional wikilinks in `related` fields
- [ ] **Step 4: Update Index** — Add all new pages to `wiki/index.md`
- [ ] **Step 5: Log the Ingest** — Append entry to `wiki/log.md` with created/updated pages, findings, and next steps
- [ ] **Run Verification Checklist** — Verify all 12 items before considering the ingest complete

---

## Resources

- **Design Spec:** See `.superpowers/sdd/2026-07-17-bazel-llm-wiki-design.md` for architecture overview
- **CLAUDE.md:** Master schema and principles
- **Index:** `wiki/index.md` — catalog of all pages
- **Log:** `wiki/log.md` — activity timeline

---

**Last Updated:** 2026-07-18  
**Status:** Active Skill
