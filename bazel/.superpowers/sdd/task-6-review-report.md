# Task 6 Review Report: `.claude/skills/query.md` Workflow Skill

**Review Date:** 2026-07-18  
**File:** `.claude/skills/query.md`  
**Commit:** cf7d198809351a37808c11d718ce29d10b6acea1  
**Verdict:** **APPROVED**

---

## Executive Summary

The Query Skill file meets all specification requirements. It is a comprehensive, well-structured 537-line workflow guide that systematically teaches how to search the wiki, synthesize answers from multiple pages, and contribute valuable results back into the knowledge base. The file is production-ready.

---

## Detailed Verification Checklist

### 1. Content Sections (5 Required)

| Section | Lines | Status | Notes |
|---------|-------|--------|-------|
| **Header** | 3–19 | ✓ PASS | Includes Purpose, When to Use, Expected Time (15–45 min), Skill Level (Intermediate) |
| **Workflow** | 22–219 | ✓ PASS | 5 distinct steps with clear outputs; 197 lines of detailed guidance |
| **Verification Checklist** | 221–239 | ✓ PASS | 8 checklist items for post-query verification |
| **Query Examples** | 243–410 | ✓ PASS | 3 diverse scenarios: simple factual, synthesis with filing, troubleshooting |
| **Guidelines** | 414–509 | ✓ PASS | 7 main guidelines + 6 common pitfalls with fixes |

**Result:** All 5 sections present and substantial. ✓

---

### 2. File Size

- **Requirement:** ≥ 120 lines
- **Actual:** 537 lines
- **Result:** PASS ✓

---

### 3. Workflow Structure (5 Steps)

| Step | Title | Lines | Clear Output? | Notes |
|------|-------|-------|---------------|-------|
| 1 | Search the Index | 24–66 | ✓ YES | Output: candidate pages, status, cross-reference awareness |
| 2 | Read Relevant Pages | 69–105 | ✓ YES | Output: detailed notes, cross-refs, contradictions, evidence |
| 3 | Synthesize & Answer | 96–145 | ✓ YES | Output: well-structured answer with citations, gaps flagged |
| 4 | File if Valuable | 148–181 | ✓ YES | Output: decision made; either no filing or new page with metadata |
| 5 | Log the Query | 184–230 | ✓ YES | Output: log entry appended to wiki/log.md |

**Result:** All 5 steps present, distinct, and actionable. ✓

---

### 4. Verification Checklist (7–8 Items)

**Required Items:**
- [ ] ✓ Question understood clearly
- [ ] ✓ Index searched thoroughly
- [ ] ✓ All candidate pages read
- [ ] ✓ Cross-references followed
- [ ] ✓ Answer is accurate and cited
- [ ] ✓ Contradictions and gaps flagged
- [ ] ✓ Decision made on filing
- [ ] ✓ Log entry appended

**Result:** 8 items present (lines 237–251). All items are actionable and testable. ✓

---

### 5. Query Examples (3 Diverse Scenarios)

| Example | Type | Pages Consulted | Decision | Outcome | Realism |
|---------|------|---|----------|---------|---------|
| 1 | Simple Factual | 1 | No filing | Direct citation | ✓ High (common scenario) |
| 2 | Synthesis | 3 | File new page | New page created with proper metadata | ✓ High (realistic filing logic) |
| 3 | Troubleshooting | 3 | No filing | Diagnostic guidance | ✓ High (practical approach) |

**Example 1 - "What is a target in Bazel?"**
- Searches index, finds direct match
- Shows when NOT to file (existing coverage)
- Concise log entry

**Example 2 - "py_library vs py_binary in Python monorepos"**
- Synthesis across 3 pages
- Reveals new pattern not explicitly covered elsewhere
- Creates new page with full metadata (title, category, level, status, sources, related)
- Shows index and cross-reference updates
- Proper log entry

**Example 3 - "Why is my Bazel build slow?"**
- Troubleshooting approach with diagnostic steps
- References multiple pages for different issues
- Shows practical command-line examples
- Appropriate decision not to file (reuse of existing guidance)
- Confidence caveat included

**Result:** All 3 examples present and realistic. ✓

---

### 6. Guidelines Section

**Main Guidelines (7):**
1. Start with the Index, Always (consistency, orientation)
2. Follow the Web of Links (web-of-knowledge principle)
3. Pay Attention to Page Status (reliability assessment)
4. Surface Contradictions & Gaps Explicitly (transparency)
5. Cite Generously (traceability)
6. Consider Filing Valuable Synthesis (compounding the wiki)
7. Log Every Query (activity timeline)

**Common Pitfalls & Fixes (6):**
1. Ignoring the Index → Always start with index
2. Not Following Related Links → Check `related` field
3. Citing Seedling Pages Without Caveats → Note page status
4. Creating a New Page When You Shouldn't → Ask "does existing content explain this?"
5. Forgetting to Update Index and Log → Always update both
6. Blending Your Own Opinions → Cite wiki, mark opinions secondary

**Total Guidelines Content:** 7 main + 6 pitfalls = 13 actionable guidance items

**Result:** Guidelines section is thorough, practical, and covers both correct practices and common mistakes. ✓

---

### 7. Markdown Formatting

| Aspect | Status | Examples |
|--------|--------|----------|
| Heading hierarchy (H1→H4) | ✓ PASS | Proper nesting throughout |
| Bold emphasis | ✓ PASS | **Goal:**, **Actions:**, **Output of Step:** |
| Bullet lists | ✓ PASS | Consistent bullet usage |
| Numbered lists | ✓ PASS | Step numbering, checklist items |
| Code blocks | ✓ PASS | Bash examples, markdown templates |
| Wikilinks | ✓ PASS | `[[concepts/fundamentals/rules]]` format |
| Horizontal rules | ✓ PASS | `---` separators between sections |
| Tables | ✓ PASS | Log entry format table |

**Result:** Markdown formatting is correct and readable. ✓

---

### 8. No Placeholder Text

- No "TODO" markers
- No "FIXME" comments
- No "[FILL IN]" or "[TO BE ADDED]" sections
- All examples include complete, realistic outputs
- All guidelines are specific and actionable

**Result:** No placeholder text detected. ✓

---

### 9. Examples Show Realistic Outcomes

**Example 1 Outcomes:**
- Identifies a matching page in the index
- Reads the page + related pages
- Provides answer with proper citations
- Correctly decides NOT to file (existing coverage)
- Logs query with appropriate confidence level

**Example 2 Outcomes:**
- Identifies synthesis opportunity (comparison not explicitly in existing pages)
- Determines actionable newness (py_library vs py_binary guidance for monorepos)
- Creates new page with complete metadata (title, category, level, status, sources)
- Updates cross-references in both directions
- Updates index with new entry
- Logs query with page creation record

**Example 3 Outcomes:**
- Demonstrates diagnostic methodology (profile → identify causes → fix)
- Shows practical commands with real-world bottlenecks
- Includes caveats ("Specific optimization depends on *your* bottleneck")
- Correctly decides NOT to file (reuse of existing guidance, no new synthesis)
- Logs query with gap identification (real-world case study lacking)

**Result:** Examples are realistic and model good decision-making. ✓

---

### 10. Commit Information

**Commit Hash:** cf7d198809351a37808c11d718ce29d10b6acea1  
**Message:** `docs: add query skill for answering questions against the wiki`  
**Body:**
```
Documents step-by-step workflow for querying wiki: search index, read pages,
synthesize answer, file if valuable, log entry. Includes verification checklist,
examples, and guidelines.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
```

**Checks:**
- ✓ Commit message format correct (`docs: ...`)
- ✓ Co-Author trailer present with correct format
- ✓ Commit body describes changes accurately
- ✓ File addition logged (536 insertions into .claude/skills/query.md)

**Result:** Commit information is correct. ✓

---

## Summary of Findings

| Item | Status | Evidence |
|------|--------|----------|
| All 5 sections present | ✓ PASS | Header, Workflow (5 steps), Checklist, Examples (3), Guidelines |
| File ≥ 120 lines | ✓ PASS | 537 lines |
| Workflow 5 steps | ✓ PASS | Search, Read, Synthesize, File, Log |
| Verification checklist 7–8 items | ✓ PASS | 8 checklist items |
| 3 diverse query examples | ✓ PASS | Simple, synthesis+filing, troubleshooting |
| Guidelines practical | ✓ PASS | 7 main + 6 pitfalls, all actionable |
| Markdown formatting correct | ✓ PASS | Proper headings, lists, code, wikilinks |
| No placeholder text | ✓ PASS | All content complete and specific |
| Examples realistic | ✓ PASS | Model good decision-making and outcomes |
| Commit proper | ✓ PASS | Message format, Co-Author trailer |

---

## Additional Observations

### Strengths

1. **Comprehensive Scope** — The skill covers the entire query lifecycle (search → synthesize → file → log), not just the answer-generation part.

2. **Real Decision Framework** — Example 2 shows when to file a new page (synthesis with new pattern) vs. Example 1 showing when not to (direct citation). This teaches judgment, not just mechanics.

3. **Cross-Reference Discipline** — The workflow explicitly emphasizes the `related` field and bidirectional linking, reinforcing the "web of knowledge" principle from CLAUDE.md.

4. **Uncertainty Handling** — Guidelines on surfacing contradictions, gaps, and seedling-page caveats model epistemic humility. Example 3 includes a confidence caveat.

5. **Practical Tooling** — Log format is grep-able (`## [YYYY-MM-DD] query |`), frontmatter schema is detailed, and filing logic includes updating index and cross-references.

6. **Common Pitfalls Section** — Beyond "how to do it right," the file anticipates 6 common mistakes and how to avoid them. This is educationally valuable.

7. **Quick Reference Checklist** — Lines 511–520 provide a condensed 5-step + verification quick reference for users in a hurry.

### Minor Notes

- The skill assumes familiarity with the wiki structure (categories, levels, frontmatter schema). This is documented in CLAUDE.md and is appropriate.
- The log format assumes a `wiki/log.md` file exists. This is managed by the ingest and lint workflows.
- All wikilink examples use realistic page paths that align with CLAUDE.md's folder structure.

---

## Verdict

**APPROVED**

The `.claude/skills/query.md` file meets all specification requirements. It is well-structured, comprehensive, pedagogically sound, and production-ready. The workflow is clear, the examples are realistic, the guidelines are practical, and the commit is properly formatted.

**Recommendation:** Deploy and use this skill for all query workflow needs. Update `wiki/log.md` consistently as queries are executed.

---

**Report Generated:** 2026-07-18  
**Reviewer:** Claude Code Agent  
**Status:** Ready for Production
