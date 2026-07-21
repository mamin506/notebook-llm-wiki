# Task 6: Query Skill Implementation Report

**Date:** 2026-07-18  
**Task:** Write `.claude/skills/query.md` - the query workflow skill  
**Status:** COMPLETE ✓

---

## Summary

Successfully implemented the Query workflow skill documentation. The file provides comprehensive step-by-step guidance for asking questions against the wiki and synthesizing answers from existing pages.

---

## File Created

**Path:** `.claude/skills/query.md`  
**Lines:** 536 (requirement: minimum 120)  
**Status:** Ready for use

---

## Content Verification

All required sections implemented:

### 1. Header Section ✓
- **Purpose:** Explains the skill's goal (systematically search, synthesize, file valuable results)
- **When to Use:** 5 clear use cases listed
- **Expected Time:** Time estimates provided (15-45 minutes based on query complexity)
- **Skill Level:** Marked as Intermediate

### 2. Workflow (5 Steps) ✓
- **Step 1: Search the Index** — Find relevant pages in wiki/index.md
  - Identify candidates by scanning categories
  - Check page status and cross-references
  - Output: list of candidate pages
  
- **Step 2: Read Relevant Pages** — Extract information and connections
  - Track cross-references systematically
  - Spot contradictions and gaps
  - Collect evidence with citations
  - Output: detailed notes, contradictions list, evidence trail
  
- **Step 3: Synthesize & Answer** — Compose coherent answer
  - Structure based on question type
  - Cite sources using wikilinks
  - Flag contradictions, gaps, uncertainty
  - Provide examples/concrete guidance
  - Output: clear, well-structured answer with citations
  
- **Step 4: File if Valuable** — Decide on new page creation
  - Assess value (synthesis? comparison? new pattern?)
  - Create page with proper frontmatter if valuable
  - Update index and cross-references
  - Output: either confirmation of filing or decision not to file
  
- **Step 5: Log the Query** — Record for future reference
  - Append to wiki/log.md with specific format
  - Include question, pages, answer summary, gaps
  - Use consistent formatting for grep-ability
  - Output: new log entry with complete metadata

### 3. Verification Checklist ✓
8 verification items covering:
- Question understanding
- Index search thoroughness
- Page reading completeness
- Cross-reference following
- Answer accuracy and citation
- Contradiction/gap flagging
- Filing decision made
- Log entry appended

### 4. Query Examples (3 Examples) ✓

**Example 1: Simple Factual Question**
- Question: "What is a target in Bazel?"
- Type: Direct factual lookup
- Process: Search index → read one page → answer directly → no file → log
- Shows: When NOT to file (already covered)

**Example 2: Synthesis Question (Requires New Page)**
- Question: "What's the difference between py_library and py_binary, and when to use each?"
- Type: Comparison requiring synthesis
- Process: Search index → read multiple pages → synthesize comparison → CREATE new page → log
- Shows: How to identify valuable synthesis and file it properly
- Result: New page `languages/python-library-vs-binary.md` with proper links

**Example 3: Troubleshooting Question**
- Question: "Why is my Bazel build slow? How do I optimize it?"
- Type: Diagnostic/troubleshooting with multiple steps
- Process: Search index → read troubleshooting pages → synthesize diagnostic approach → no file (reuses existing) → log
- Shows: How to structure troubleshooting guidance with prioritization

### 5. Guidelines (7+ Guidelines) ✓

Seven practical guidelines for effective querying:
1. **Start with the Index, Always** — Emphasizes index as compass, not filesystem search
2. **Follow the Web of Links** — Shows importance of related field and cross-references
3. **Pay Attention to Page Status** — How to assess reliability (seedling vs mature)
4. **Surface Contradictions & Gaps Explicitly** — When/how to flag uncertainty
5. **Cite Generously** — Using wikilinks, page sections, and status in citations
6. **Consider Filing Valuable Synthesis** — Types of content worth capturing as new pages
7. **Log Every Query** — Importance of activity log for compounding knowledge

### Additional Sections ✓

- **Common Pitfalls & How to Avoid Them** (6 pitfalls covered)
  - Ignoring the index
  - Not following related links
  - Citing seedling pages without caveats
  - Creating pages unnecessarily
  - Forgetting to update index/log
  - Blending opinions without marking them
  
- **Query Workflow Checklist** — Quick reference for 5 steps + verification
  
- **Resources** — Links to related files (design spec, CLAUDE.md, sibling skills)

---

## Commit Details

**Commit Hash:** cf7d198  
**Message:**
```
docs: add query skill for answering questions against the wiki

Documents step-by-step workflow for querying wiki: search index, read pages,
synthesize answer, file if valuable, log entry. Includes verification checklist,
examples, and guidelines.

Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
```

**Changes:**
- 1 file created: `.claude/skills/query.md`
- 536 lines added
- Properly formatted with YAML frontmatter conventions consistent with ingest.md

---

## Relationship to Design Spec

File implements section 6.2 (QUERY workflow) from the design specification (`2026-07-17-bazel-llm-wiki-design.md`):

| Spec Section | Coverage |
|---|---|
| Workflow steps (1-5) | ✓ Complete, with detailed actions for each step |
| Query goals | ✓ Documented in Step 3 synthesis guidance |
| Expected output | ✓ Specified for each workflow step |
| Bazel-specific considerations | ✓ Embedded in guidelines (e.g., terminology consistency) |
| Verification checklist | ✓ 8-item checklist covering quality gates |
| Examples | ✓ 3 examples with different scopes and complexity |
| Common patterns | ✓ 7+ guidelines covering effective practice |

---

## File Structure & Quality

- **Format:** Markdown with consistent heading hierarchy
- **Navigation:** Clear section numbering, table of contents flow
- **Readability:** Short paragraphs, bullet points, code blocks, examples
- **Actionability:** Every section includes concrete "Actions" and "Output"
- **Consistency:** Follows ingest.md style and structure for coherence across skills
- **Searchability:** Grep-friendly log format, consistent terminology
- **Cross-references:** Links to CLAUDE.md, design spec, related skills, index, log

---

## Next Steps

The query.md skill is now ready for use. It can be:
1. Referenced in CLAUDE.md as part of the wiki's operational manual
2. Used when addressing questions about Bazel (direct users to the skill)
3. Extended with Bazel domain-specific queries as the wiki grows
4. Paired with lint.md (health checks) and ingest.md (source integration) to form complete wiki workflow

---

## Sign-Off

✓ File created and committed  
✓ All required content sections present and complete  
✓ Exceeds minimum length requirement (536 lines > 120)  
✓ Format consistent with existing skills (ingest.md)  
✓ Aligned with design specification section 6.2  
✓ Ready for integration into wiki workflow documentation

**Ready for production use.**
