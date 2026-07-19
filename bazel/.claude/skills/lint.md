# Lint Skill: Periodic Wiki Health Checks

## Header

**Purpose:** Systematically audit the wiki for inconsistencies, stale content, missing cross-references, gaps, and orphaned pages. Each lint pass ensures the wiki remains coherent, up-to-date, and well-connected. Lint is maintenance work—keeping the knowledge base healthy and discoverability high.

**When to Use:**
- Monthly health check (even if no major ingests)
- After large ingest batches (5+ sources or 10+ pages modified)
- When you notice inconsistencies or want a comprehensive audit
- Before archiving or refactoring sections
- When preparing the wiki for reference or sharing

**Expected Time:** 45–120 minutes depending on wiki size and complexity (small wikis: 45–60 minutes; mature wikis with 50+ pages: 90–120 minutes)

**Skill Level:** Intermediate (requires understanding wiki structure, cross-references, and frontmatter conventions)

---

## Workflow

### Step 1: Check for Contradictions

**Goal:** Identify pages claiming opposite things and resolve discrepancies.

**Actions:**

1. **Scan for conflicting claims**
   - Read pages that cover similar topics or the same concept from different angles
   - Example: Compare `concepts/fundamentals/rules.md` with `concepts/advanced/macros.md` if both discuss rule-like behavior
   - Look for statements like "X always happens" vs. "X never happens" or different procedural steps
   - Check version-specific claims (e.g., "available in Bazel 5.0+" vs. "available in Bazel 3.0+")

2. **Flag contradictions with their sources**
   - When you find a contradiction, note:
     - The two conflicting claims
     - Which pages contain them
     - Which sources contributed to each page (from the `sources` field)
   - Example: "Page A (from source-v1.md) says aspects run during analysis; Page B (from source-v2.md) says aspects run during action"

3. **Determine the cause**
   - Is the contradiction due to:
     - **Different Bazel versions?** (e.g., behavior changed between versions)
     - **Different contexts?** (e.g., both true but in different scenarios)
     - **Source error?** (e.g., one source is outdated or wrong)
     - **Wiki error?** (e.g., you misinterpreted the source)

4. **Resolve contradictions**
   - **If both are valid:** Update one or both pages to clarify context. Add a "Note" or "Differs From" section explaining the discrepancy.
     ```markdown
     **Note on Aspect Timing:** Aspects can run during different phases depending on configuration.
     See [[concepts/advanced/aspects-phases]] for detailed explanation of timing.
     ```
   - **If one is newer/better:** Update the older page to reference the newer one.
   - **If it's a source error:** Update the page, note the error, and flag for verification.
   - **If it's ambiguous:** Create a new synthesis page (e.g., `concepts/advanced/macros-vs-rules-detailed.md`) that clarifies both perspectives.

5. **Update or create cross-references**
   - After resolving a contradiction, ensure both pages link to each other in `related` fields
   - If you create a synthesis page, add it to both pages' `related` fields

**Output of Step 1:**
- List of contradictions found and how each was resolved
- Updated pages with clarifications or cross-references
- Possibly a new synthesis page if the contradiction was complex

---

### Step 2: Spot Stale Claims

**Goal:** Find facts that have been superseded by newer sources or that need verification.

**Actions:**

1. **Review pages with old `last_updated` dates**
   - Scan `wiki/` directory for pages with `last_updated` > 3 months ago
   - Priority: Check pages in reference/, languages/, tools/, and troubleshooting/ (these are most likely to become stale)
   - Example: `last_updated: "2025-12-01"` is probably stale if today is July 2026

2. **Check for version-specific claims without version info**
   - Look for statements like "Use this Bazel feature..." without specifying what Bazel version is required
   - Example: "The `--define` flag controls..." might be outdated or version-specific
   - If you find this, add or update version info in the page

3. **Cross-reference with newer sources**
   - Check the wiki log (`wiki/log.md`) to see what newer sources have been ingested
   - If newer sources cover the same topic as an old page, read both
   - Example: If a page about Python Bazel rules was last updated 6 months ago, but you ingested a newer Python guide last month, compare them

4. **Look for "TODO" or "Note: verify" markers**
   - Search pages for TODO, FIXME, or "verify with newer source" comments
   - These are flags left by previous ingests indicating uncertainty
   - Decide: Update with new source, or mark for follow-up

5. **Check claim currency against official docs**
   - For reference pages (CLI reference, built-in rules, etc.), skim official Bazel docs to spot obvious differences
   - Example: If a page says "Use `bazel build --foo`" but official docs say the flag is now `--bar`, update it

6. **Update stale pages**
   - Update `last_updated` to today's date if you edited the page
   - Update `status` if content improved (e.g., growing → mature)
   - Add newer sources to `sources` list if you used them to update the page
   - Add a note in the log about what was updated

**Output of Step 2:**
- List of stale pages found and what was updated
- Updated pages with current information or version-specific notes
- Flagged pages that need new sources (mark in log for follow-up)

---

### Step 3: Find Orphan Pages

**Goal:** Identify pages with no inbound links and integrate or remove them.

**Actions:**

1. **Search for unlinked pages**
   - Read `wiki/index.md` to see all pages listed
   - For each page, search the wiki folder to check:
     - Do other pages link to it? (Check body text for `[[page-name]]` wikilinks)
     - Is it listed in any `related` fields? (Check frontmatter of other pages)
     - Is it the index or log? (These are OK to not be linked)
   - Pages with zero inbound links (except the index) are orphans

2. **Categorize orphans**
   - **Foundational but unlinked:** A page should be linked but isn't due to oversight
   - **Too specialized:** A page is so specific that nothing else references it; may be truly orphan or may need integration
   - **Incomplete stub:** A page was created but never developed; perhaps it should be deleted or merged
   - **Truly obsolete:** A page is outdated and should be removed

3. **Fix orphans**
   - **For foundational pages:** Add bidirectional links to related pages. Update `related` fields.
     - Example: If `concepts/fundamentals/labels.md` is orphaned, add it to `related` fields in `targets.md` and `rules.md`
   - **For specialized pages:** Consider merging into a parent page if it's short, or create a parent/index page that references it
     - Example: If you have 5 pages about performance but no `troubleshooting/performance-index.md`, create one
   - **For incomplete stubs:** Either complete the page (mark for follow-up) or delete it
   - **For obsolete pages:** Delete and note in the log

4. **Update index if needed**
   - If you delete a page, remove it from `wiki/index.md`
   - If you significantly restructured orphans, update index organization

**Output of Step 3:**
- List of orphaned pages found
- List of how each was fixed (linked, merged, deleted, or marked for follow-up)
- Updated `related` fields in affected pages
- Updated `wiki/index.md` if pages were deleted

---

### Step 4: Identify Gaps

**Goal:** Find important concepts mentioned in multiple pages but lacking their own dedicated page.

**Actions:**

1. **Scan for "See [[page]]" or "See also:" references that point to missing pages**
   - Search pages for patterns like:
     - `[[concepts/missing-concept]]` (wikilinks that don't exist)
     - "For more on X, see [link]" where link is broken
     - "Discussed in [[page]] but not in detail here"
   - These are hints that a concept needs its own page

2. **Look for repeated mentions of the same concept across different pages**
   - Example: If `targets.md`, `rules.md`, and `artifacts.md` all mention "the dependency graph" but there's no `concepts/fundamentals/dependencies.md`, that's a gap
   - Count how many pages mention a concept; if 3+ pages reference it, it deserves its own page

3. **Check for missing cross-cutting topics**
   - Review `tags` across pages (e.g., `#monorepo`, `#performance`, `#debugging`)
   - Are there index pages or synthesis pages for each tag?
   - Example: If many pages use `#monorepo`, is there a `patterns/monorepo-index.md` or `patterns/monorepo-layout.md` that ties them together?

4. **Review open questions in the log**
   - Read `wiki/log.md` and look for entries with "Open questions:" or "Gaps identified:"
   - Prioritize gaps that appear in multiple log entries

5. **Create new pages or flag for follow-up**
   - **Create immediately** if the gap is simple (< 1 hour to write a decent seedling page)
     - Example: You found "the dependency graph" is mentioned 5 times but has no page. Create `wiki/concepts/fundamentals/dependencies.md` as a seedling page with basic info and links to relevant pages
   - **Flag for follow-up** if the gap requires a new source to understand well
     - Example: Multiple pages mention "Bazel's remote execution model" but it's not explained. Flag in the log: "Need source on remote execution"
   - Either way, add the gap to the lint log entry so it's recorded

6. **Update cross-references**
   - After creating a new page, add it to `related` fields of pages that mentioned the concept
   - Update the index with the new page

**Output of Step 4:**
- List of gaps found
- 0–3 new seedling pages created to fill critical gaps
- List of gaps flagged for follow-up (in log)
- Updated index and cross-references

---

### Step 5: Check Cross-References

**Goal:** Ensure pages that should be linked are linked, and `related` fields are complete and bidirectional.

**Actions:**

1. **Verify bidirectional links**
   - For each page, check its `related` field
   - Then check the pages it links to; they should have reciprocal links back
   - Example: If `targets.md` lists `rules.md` in `related`, then `rules.md` should list `targets.md` in its `related`

2. **Identify missing cross-references**
   - For each major concept page, ask: "What other pages should know about this?"
   - Example: If you have `patterns/monorepo-layout.md`, should it link to `languages/python.md`, `languages/java.md`, etc.? (Probably yes)
   - Scan related pages and add missing links
   - Check category boundaries: Does a concepts/ page link to patterns/? Do reference/ pages link to troubleshooting/?

3. **Fix wikilink syntax errors**
   - Search for malformed wikilinks:
     - `[[page-with-spaces]]` (should be hyphens or underscores, not spaces)
     - `[[concepts/page]]` pointing to a file that doesn't exist
     - `[[page.md]]` (should not have .md extension)
   - Correct these so links are valid

4. **Update `related` fields**
   - Add missing links to `related`
   - Ensure each link is reciprocated
   - Keep lists organized (alphabetical or logical order)

5. **Check in-page wikilinks**
   - Search page bodies for wikilinks
   - Ensure they point to valid pages
   - Verify they're contextually relevant (not random links)

**Output of Step 5:**
- List of bidirectional link issues found and fixed
- List of missing cross-references added
- List of wikilink syntax errors corrected
- Updated `related` fields across affected pages

---

### Step 6: Scan for Incomplete Sections

**Goal:** Find pages with TODO markers, incomplete explanations, or sections marked "TBD" and decide: complete or flag for follow-up.

**Actions:**

1. **Search for TODO/FIXME/TBD markers**
   - Use a text search for patterns:
     - `TODO:`
     - `FIXME:`
     - `TBD`
     - `[incomplete]`
     - `[needs work]`
     - `...more to come...`
   - Record which pages have these markers

2. **Review flagged sections**
   - Read each incomplete section in context
   - Assess: Can you complete it now, or does it need a new source?

3. **Complete if possible**
   - If you can write a good completion from your knowledge or existing sources:
     - Complete the section
     - Update `last_updated` and `status`
     - Remove the TODO marker
   - Example: A page might say "TODO: Add examples of using `bazel build` with custom flags". You can add a quick example from your knowledge.

4. **Flag for follow-up if needed**
   - If completion requires research or a new source:
     - Leave the TODO marker
     - Update it to be more specific: `TODO: Add examples from rules_python guide (need source on rules_python)`
     - Add the gap to the lint log entry under "Flagged for follow-up"

5. **Check for vague or incomplete explanations**
   - Look for sections that seem thin or unfinished even without a TODO marker
   - Examples:
     - "Macros are useful" (but no explanation of why or when)
     - A section with one paragraph and no examples
     - A reference page missing common use cases
   - Flag these for expansion in future ingests

**Output of Step 6:**
- List of incomplete sections found
- Number of sections completed during this lint pass
- List of sections flagged for follow-up (in log)
- Updated pages with completions

---

### Step 7: Log the Lint Pass

**Goal:** Record all findings, fixes, and recommendations for future reference.

**Actions:**

1. **Open `wiki/log.md`**
   - This is where you record all lint activity

2. **Append a new lint entry at the end**
   - Use this format:
   ```markdown
   ## [YYYY-MM-DD] lint | [Type of Lint Pass]
   
   **Contradictions found:** N (summary of each)
   **Stale claims:** N (list of updated pages)
   **Orphan pages:** N (how fixed)
   **Gaps identified:** N (created X new pages, flagged Y for follow-up)
   **Missing cross-refs:** N (relationships added)
   **Incomplete sections:** N (completed X, flagged Y)
   
   **Summary:** [1-2 sentences on overall wiki health]
   **Recommendations:** [What to prioritize next; suggested follow-up sources]
   **Pages modified:** [[page1]], [[page2]], etc.
   ```

3. **Be specific with counts and details**
   - Don't just say "Fixed contradictions"; say "Found 2 contradictions: X vs. Y (resolved with note in [[page]]); A vs. B (created synthesis page [[new-page]])"
   - List actual page paths, not just categories
   - Include actionable recommendations for future work

4. **Example Log Entry:**
   ```markdown
   ## [2026-07-28] lint | Monthly health check after Python ingest batch
   
   **Contradictions found:** 2
   - "py_library vs py_compiled_library: use py_library always" vs. "py_compiled_library is faster for large projects" — resolved with context note in [[languages/python]] clarifying when each is appropriate
   - "External Python deps go in BUILD" vs. "External Python deps go in WORKSPACE" — found source version difference; updated [[patterns/dependency-management]] with version note (Bazel 4.0+)
   
   **Stale claims:** 3
   - [[languages/python]] last updated 2026-03-15; updated with examples from latest ingest
   - [[tools/gazelle]] last updated 2026-02-01; verified still current but added note on newer gazelle extensions
   - [[troubleshooting/cache-issues]] last updated 2025-11-30; expanded with new remote cache configuration options
   
   **Orphan pages:** 1
   - [[reference/deprecated-flags]] was created but never linked; integrated into [[reference/cli-reference]] and marked as section
   
   **Gaps identified:** 3
   - "Monorepo layout" is mentioned in 4 pages but no detailed guide; created [[patterns/monorepo-layout]] as seedling
   - "Python testing strategies" mentioned in [[languages/python]] but no dedicated page; flagged for follow-up source
   - "BUILD file macros" mentioned frequently; created [[concepts/advanced/build-macros]] cross-reference
   
   **Missing cross-refs:** 5
   - Added [[patterns/monorepo-layout]] to related fields in [[languages/python]], [[languages/java]], [[patterns/dependency-management]]
   - Added bidirectional links between [[tools/gazelle]] and [[reference/builtin-rules]]
   - Fixed 2 broken wikilinks: `[[targets]]` → `[[concepts/fundamentals/targets]]`
   
   **Incomplete sections:** 2
   - Completed "Examples" section in [[patterns/monorepo-layout]] with code snippet from source
   - Flagged "TODO: cross-platform builds" in [[patterns/cross-platform-builds]] for follow-up (needs C++ and iOS source)
   
   **Summary:** Wiki is in good health. 3 pages updated, 2 new seedling pages created. Cross-reference web is strengthening. Main gaps are advanced topics (custom rules, aspects) and language-specific testing.
   
   **Recommendations:** Prioritize ingesting source on Python testing strategies and custom rules development. Consider cross-platform Bazel guide. Monorepo layout is now stronger but could use advanced examples.
   
   **Pages modified:** [[languages/python]], [[tools/gazelle]], [[troubleshooting/cache-issues]], [[patterns/monorepo-layout]], [[concepts/advanced/build-macros]], [[reference/cli-reference]]
   ```

**Output of Step 7:**
- New entry appended to `wiki/log.md`
- Clear record of wiki health, fixes applied, and recommendations for future work
- Guidance for planning next ingests or targeted improvements

---

## Verification Checklist

Before considering a lint pass complete, verify the following 10 items:

- [ ] **All pages checked for contradictions** — Scanned major topic pages (concepts, reference, patterns) for conflicting claims; all contradictions either resolved or documented with sources

- [ ] **Stale claims identified and updated** — Checked pages with `last_updated` > 3 months; reviewed against newer sources; updated or flagged for verification

- [ ] **Orphan pages found and fixed** — Identified all pages with no inbound links; linked, merged, or deleted as appropriate; updated index if deletions made

- [ ] **Gaps documented** — Identified concepts mentioned in multiple pages but lacking dedicated pages; created seedling pages for critical gaps; flagged others for follow-up

- [ ] **Cross-references are bidirectional** — Verified that if page A links to B in `related`, B links back to A; added missing reciprocal links

- [ ] **Incomplete sections resolved** — Searched for TODO/FIXME markers; completed what could be done; flagged remainder for follow-up with specific notes

- [ ] **Wikilinks are valid** — Corrected broken wikilinks; verified all `[[page]]` references point to existing files with correct syntax (no .md extension, hyphens not spaces)

- [ ] **No regression** — Verified that fixes didn't introduce new orphans or contradictions (spot-check related pages after major changes)

- [ ] **Log entry created** — Appended lint entry to `wiki/log.md` with specific counts, pages modified, and recommendations

- [ ] **Status fields updated** — Updated `last_updated` on all modified pages; reviewed and bumped `status` if appropriate (e.g., growing → mature if gaps filled)

---

## Lint Cadence & Triggers

### Monthly Health Check

**Schedule:** First week of each month (or every 4 weeks if on a different calendar)

**Scope:** Full lint pass
- Check all categories for contradictions
- Scan for stale claims
- Find orphans and gaps
- Verify cross-references
- Clean up incomplete sections

**Expected time:** 60–90 minutes for a mature wiki (30+ pages)

**Goal:** Keep the wiki fresh, cohesive, and well-connected between major ingest cycles

---

### Post-Ingest Lint

**Trigger:** After ingesting 5+ sources or modifying 10+ pages

**Scope:** Targeted lint focusing on new/changed content
- Check new pages for contradictions with existing content
- Verify new pages are linked (not orphaned)
- Ensure `related` fields are populated and bidirectional
- Update index with new pages
- Spot obvious gaps introduced by new content

**Expected time:** 30–45 minutes

**Goal:** Ensure new content integrates smoothly; catch integration issues early

---

### Periodic Deep Lint

**Trigger:** Quarterly (every 3 months) or when preparing for major refactoring/reorganization

**Scope:** Comprehensive audit plus structural review
- Full monthly lint pass
- Review overall category structure: are categories balanced? Should any be split or merged?
- Check tag consistency: do all `#monorepo` pages actually relate to monorepos?
- Verify index organization and accuracy
- Identify if any sections should be split (e.g., `patterns/` growing too large)
- Look for opportunities to create synthesis pages (e.g., comparison of two approaches)

**Expected time:** 120–180 minutes

**Goal:** Maintain long-term structure and quality; prevent wiki from becoming disorganized

---

## Example: Lint Pass After Python Ingest

**Scenario:** You just completed a major ingest batch (3 Python sources, 8 pages modified). Time for a post-ingest lint.

### Step 1: Check for Contradictions
- Compare new `[[languages/python]]` content with existing `[[concepts/fundamentals/rules]]`
- **Finding:** New content says "Use py_library for all Python packages" but existing patterns page says "Use py_binary for executables". Not a contradiction—clarified in related fields.
- **Finding:** New source claims "pytest integration via rules_python" but old patterns page says "no pytest support yet". Source was newer; updated patterns page and added new source to `sources`.
- **Resolution:** Updated [[patterns/test-strategy]] to reference [[languages/python]] section on pytest

### Step 2: Spot Stale Claims
- Checked `last_updated` on Python-related pages
- **Finding:** [[troubleshooting/build-failures]] mentions "py_library doesn't support pytest" but new source shows full pytest support. Updated with new examples.
- **Finding:** [[reference/builtin-rules]] lists `py_library` but no detail on `py_test`. Added section with reference to [[languages/python]].
- **Resolution:** Updated two reference pages with new info; marked as growing status

### Step 3: Find Orphan Pages
- Reviewed new pages from ingest: [[languages/python/deps]], [[languages/python/testing]], [[languages/python/monorepos]]
- **Finding:** [[languages/python/deps]] exists but no other pages link to it yet
- **Resolution:** Added to `related` in [[patterns/dependency-management]]; added reciprocal link in [[languages/python/deps]]

### Step 4: Identify Gaps
- Scanned new pages for mentioned-but-unexplained concepts
- **Finding:** Python pages mention "virtual environments" and "toolchains" but no dedicated pages
- **Finding:** "Python monorepos" mentioned 3 times; created seedling [[patterns/python-monorepos]] with links to [[patterns/monorepo-layout]]
- **Resolution:** Created 1 new page; flagged "virtual environments" for future ingest

### Step 5: Check Cross-References
- Verified all new pages have `related` fields populated
- **Finding:** [[languages/python]] doesn't link to [[languages/java]] or other languages
- **Finding:** [[patterns/dependency-management]] updated but didn't add link back to [[languages/python/deps]]
- **Resolution:** Added reciprocal links; each language page now links to others in `related`

### Step 6: Scan for Incomplete Sections
- Searched new pages for TODOs
- **Finding:** [[languages/python/testing]] has "TODO: Add example of custom test rule"
- **Resolution:** Completed with example from source; removed TODO

### Step 7: Log the Lint Pass
```markdown
## [2026-07-28] lint | Post-ingest Python sources

**Contradictions found:** 1 (version discrepancy resolved; updated [[patterns/test-strategy]])
**Stale claims:** 2 ([[troubleshooting/build-failures]], [[reference/builtin-rules]] updated with pytest support)
**Orphan pages:** 1 ([[languages/python/deps]] linked to [[patterns/dependency-management]])
**Gaps identified:** 2 (created [[patterns/python-monorepos]]; flagged virtual environments for follow-up)
**Missing cross-refs:** 3 (added bidirectional links between language pages and dependency patterns)
**Incomplete sections:** 1 (completed testing examples in [[languages/python/testing]])

**Summary:** New Python content integrates well. One new synthesis page created. Wiki now has better language-to-language and language-to-patterns cross-referencing.

**Recommendations:** Prioritize source on Python virtual environments and toolchain configuration. Consider comparing Python monorepos to Java and C++ monorepos for synthesis.

**Pages modified:** [[languages/python]], [[patterns/dependency-management]], [[patterns/test-strategy]], [[troubleshooting/build-failures]], [[reference/builtin-rules]], [[languages/python/deps]], [[languages/python/testing]], [[patterns/python-monorepos]]
```

---

## Lint Tips

### 1. **Search Systematically, Not Randomly**
Use grep or search tools to find patterns rather than manually reading every page. Examples:
- Find all TODO markers: `grep -r "TODO:" wiki/`
- Find all wikilinks: `grep -r "\[\[" wiki/` to check for syntax errors
- Find old pages: `grep -r "2026-04" wiki/` to find pages last updated in April
- Find orphans: `grep -rL "\[\[" wiki/ | grep -v index.md` to find pages with no outbound links (rough approximation)

### 2. **Use the Index as Your Checklist**
Read through `wiki/index.md` line by line. For each entry:
- Does the page exist where expected?
- Is the status current? (Should seedling become growing? Should growing become mature?)
- Should this page link to others? If yes, check if it does.
This takes 15–20 minutes but is very effective for finding overview-level issues.

### 3. **Follow Cross-Reference Chains**
When you find a link or reference, follow it:
- Does page A link to B? Check if B links back to A.
- Does the `related` field contain valid pages? Click each one (or check manually).
- Do pages that should be related (e.g., two language pages) actually link to each other?
This catches reciprocal link failures and missing connections quickly.

### 4. **Tag-Based Gap Hunting**
Look for common tags across pages (e.g., `#monorepo`, `#performance`). Ask:
- Are all pages with the same tag cross-linked?
- Is there an index or synthesis page for this tag?
- Are there obvious pages missing this tag that should have it?
Example: Pages tagged `#monorepo` might not all link to each other; create a monorepo index page or update each one's `related` field.

### 5. **Version-Specific Content Audit**
Search for Bazel version mentions. For each one:
- Is it accurate? (Spot-check against official docs if uncertain)
- Are version ranges specified consistently? (E.g., "Bazel 5.0+" not just "newer Bazel")
- If multiple versions are mentioned in one page, is the relationship clear?
Example: If a page says "Feature X available in Bazel 6.0+" but also mentions "Feature Y added in 5.0", clarify the timeline.

### 6. **Contradiction Hunting: Compare Canonical Pages**
For each major concept, identify the "canonical" page (the main reference). Then:
- Scan related pages for claims that might contradict the canonical page
- Check the order of steps (if pages describe a procedure, do they agree on order?)
- Verify definitions are consistent (does every page define a term the same way?)
Example: `concepts/fundamentals/targets.md` is canonical for targets; scan rules, artifacts, and labels pages for consistent use of "target"

### 7. **Lint After Adding a New Source**
Don't wait for monthly lint. After each major ingest, spend 15–20 minutes on targeted lint:
- Do new pages orphan old pages?
- Are there new contradictions?
- Are gaps filled or new gaps created?
This prevents small issues from compounding.

---

## Common Lint Issues & Fixes

| Issue | Symptom | How to Fix |
|-------|---------|-----------|
| **Bidirectional link broken** | Page A links to B in `related`, but B doesn't link back to A | Open page B, add page A to its `related` field |
| **Orphan page** | No other pages link to a page; only the index references it | Add it to `related` fields of 2–3 related pages; ensure those pages link back |
| **Wikilink syntax error** | `[[concepts/page.md]]` or `[[Page With Spaces]]` or `[[page-that-doesnt-exist]]` | Correct to `[[concepts/page]]` (no .md, lowercase, hyphens for spaces); verify file exists |
| **Stale `last_updated`** | Page's `last_updated` is > 3 months old and may contain outdated info | Read the page; if it's still accurate, update `last_updated` to today; if it needs work, flag for follow-up source |
| **Contradictory claims in same page** | One paragraph says "X" and another says "not X" | Combine into one clear statement; if both are valid in different contexts, add "Note" section explaining difference |
| **Missing cross-category link** | Concepts page mentions a pattern but doesn't link to patterns/ category | Add wikilink to concepts page body or update `related` field in frontmatter |
| **Incomplete TODO** | Page has `TODO: [vague task]` instead of specific action | Make it specific: `TODO: Add example of [X] from [source name or topic]` |
| **Inconsistent terminology** | Page A says "targets" and page B says "build targets" for the same concept | Standardize on official Bazel term ("targets"); update page B; note mapping if source used different term |
| **Missing source attribution** | Page has no `sources` field or it's empty | Identify which raw files contributed to the page; add to `sources` list; add `last_updated` if missing |
| **Status field incorrect** | Page marked "seedling" but has multiple sources, examples, and comprehensive content | Update to "growing" or "mature"; this helps readers understand completeness |
| **Orphaned synthesis page** | Page like "Macros vs Rules" exists but doesn't link to either `macros.md` or `rules.md` | Add to `related` fields in both `macros.md` and `rules.md`; verify those pages link back |
| **Circular or redundant links** | Page A links to B, B links to C, C links back to A, but these are all saying the same thing | Consolidate: pick one canonical page; have others link to it; remove redundant mutual links |

---

## Lint Workflow Checklist (Quick Reference)

Use this as a quick reference while linting:

- [ ] **Step 1: Check for Contradictions** — Scan major pages for conflicting claims; resolve with clarifications, updates, or synthesis pages
- [ ] **Step 2: Spot Stale Claims** — Review old pages and version-specific content; update or flag for verification
- [ ] **Step 3: Find Orphan Pages** — Identify unlinked pages; add links, merge, or delete as appropriate
- [ ] **Step 4: Identify Gaps** — Find concepts mentioned but not explained; create seedling pages or flag for follow-up
- [ ] **Step 5: Check Cross-References** — Verify bidirectional links and add missing connections
- [ ] **Step 6: Scan for Incomplete Sections** — Find TODOs; complete what's possible; flag remainder for follow-up
- [ ] **Step 7: Log the Lint Pass** — Append entry to wiki/log.md with findings, fixes, and recommendations
- [ ] **Run Verification Checklist** — Verify all 10 items before considering lint pass complete

---

## Resources

- **Design Spec:** See `.superpowers/sdd/2026-07-17-bazel-llm-wiki-design.md` for architecture overview
- **CLAUDE.md:** Master schema and principles
- **Ingest Skill:** `.claude/skills/ingest.md` for source integration workflow
- **Query Skill:** `.claude/skills/query.md` for Q&A workflow
- **Index:** `wiki/index.md` — catalog of all pages
- **Log:** `wiki/log.md` — activity timeline

---

**Last Updated:** 2026-07-18  
**Status:** Active Skill
