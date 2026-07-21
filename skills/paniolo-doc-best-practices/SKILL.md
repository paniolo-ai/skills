---
name: paniolo-doc-best-practices
description: |
  Documentation standards for this project — file naming, structure, writing style, formatting, skill+doc pairs, placement, and maintenance. Use when creating, updating, reviewing, or auditing any Markdown doc in sharp-shooter-wiki `wiki/`, `README.md`, or `CONTRIBUTING.md`. Also use when deciding whether a new topic needs a doc, a skill, or both. Do NOT use for in-code JSDoc/TSDoc comments — load code-comment-best-practices instead.
license: MIT
metadata:
  version: 0.1.0
tags:
- documentation
references:
- 'wiki: sharp-shooter-wiki:doc-content → references/doc-content.md'
- 'wiki: sharp-shooter-wiki:doc-structure → references/doc-structure.md'
- 'wiki: sharp-shooter-wiki:markdown-anchor-links → references/markdown-anchor-links.md'
- 'wiki: sharp-shooter-wiki:markdown-file-naming → references/markdown-file-naming.md'
- 'wiki: sharp-shooter-wiki:markdown-required-elements → references/markdown-required-elements.md'
- 'wiki: sharp-shooter-wiki:writing-style-writing-style → references/writing-style-writing-style.md'
user-invocable: true
---

**Requires:** file-read. No terminal needed unless validating after edits.

## Full reference

[doc-content](references/doc-content.md) — load
on demand for full formatting rules, ✅/❌ examples, rationale, and edge cases.

## When invoked

**Preconditions:**

- Read the target file before modifying it.
- Check rules for repo-wide constraints before
  proceeding.

**Clarifying questions:**

- **Defaults (proceed without asking):** assume the task is creating a new wiki authoring page
  unless an existing path is mentioned; place new practice docs under
  `sharp-shooter-wiki/wiki/<category>/` unless context suggests README or CONTRIBUTING.
- **Always ask:** topic and purpose if creating and not provided; file path if
  reviewing/updating and not provided. Ask one focused question at a time.
- State assumptions when proceeding: "Creating a new wiki authoring page — let me
  know if you meant to update an existing one."

**Output format:**

- _Creating:_ full file as a single fenced Markdown block + brief bullet list
  of notable decisions.
- _Reviewing:_ structured report — **Passes** (bullets) and **Gaps** (table:
  item | issue | suggested fix). Terse.
- _Updating:_ changed sections as fenced code with one sentence per change.
- _Question-answering:_ concise prose referencing the relevant doc section.

**Error handling:**

- If the file path is needed but cannot be inferred, ask before proceeding.
- If the specified file does not exist, stop and report: "File not found: [path]."

## Quick decision trees

**Where does this doc live?**

```text
├─ General project info (setup, contributing) → Root README.md
├─ Coding standards & patterns               → sharp-shooter-wiki/wiki/<topic>/
├─ Feature or architecture deep dive         → sharp-shooter-wiki/wiki/ or repo `docs/`
├─ Code-level implementation details         → Inline JSDoc/TSDoc comments
└─ Skill definitions for AI agents           → /.agents/skills/[name]/SKILL.md
```

**Do I need a skill too?** Decision tree and full pair table:
skill-and-doc-pairing.

## Quick rules

### File naming

- Lowercase with hyphens: `doc-best-practices.md` ✅
- No underscores, dots, or PascalCase: `DocBestPractices.md` ❌

**Details:** [markdown-file-naming](references/markdown-file-naming.md)

### Required structure

Every **wiki authoring page** (`sharp-shooter-wiki/wiki/*.md`) should have:

1. **H1 title** — one per document, at the top
2. **Brief intro** — 1–3 sentences: what is this, who should read it
3. **Table of Contents** — for docs >~150 lines or with 3+ sections
4. **Explicit `<a id="...">` anchors** — for every heading in the ToC (docs only; never in
   `.agents/skills/*/SKILL.md`)
5. **`## See Also`** — at the bottom, linking to related docs (optional but preferred)

**Details:** [markdown-required-elements](references/markdown-required-elements.md),
[markdown-anchor-links](references/markdown-anchor-links.md)

### Formatting essentials

- **Headers:** ATX style (`#`), Title Case, one blank line before/after,
  unique names across the whole document
- **Lists:** hyphens (`-`), 4-space indent for nesting, `1.` for ordered
- **Code blocks:** always fenced with language specifier (` ```typescript `)
- **Section separators:** `---` between major H2 sections only
- **Line length:** target 80–100 chars for prose; skip for URLs, tables, code

**Details:**

### Writing style

- Active voice; short sentences (<25 words preferred)
- Audience: competent developer, new to this project
- Explain the "why" when rules might seem arbitrary
- Use decision trees for multi-path choices

**Details:** [writing-style-writing-style](references/writing-style-writing-style.md)

### Links

- Internal: relative paths (`react-best-practices.md`, `../README.md`)
- External: absolute URLs with descriptive anchor text (not "click here")
- Section links: `[Section Title](#anchor-id)` using explicit `<a id>` anchors

**Details:**

### Write for retrieval (qmd)

Docs are indexed by qmd; structure changes whether it surfaces your content.

- Specific, keyword-rich **H1** (it's the Title, embedded with every chunk).
- **Section liberally**; keep sections ≲900 tokens (qmd's chunk size); put the
  answer right under its heading.
- Use the **real terms/aliases** searchers query, not only paraphrases.
- Keep code blocks tight; **one concept per file** in a sensible folder.

**Details:** the qmd rationale is woven into the relevant rules — see the
**Retrieval (qmd):** notes in
[doc-structure](references/doc-structure.md).

## Validation checklist

- [ ] Filename kebab-case, no underscores
- [ ] H1 title present; exactly one per file
- [ ] Brief intro (1–3 sentences) immediately after H1
- [ ] ToC present (if >~150 lines or 3+ sections) — docs only
- [ ] `<a id="...">` anchors match all ToC entries — docs only, not skills
- [ ] ATX headers, Title Case, no skipped levels
- [ ] Code blocks fenced with language specifier
- [ ] No broken internal links
- [ ] `## See Also` at bottom with links to related docs

## Do Not

- Do not use for in-code JSDoc/TSDoc comments — load `code-comment-best-practices` instead.
- Do not use for skill file authoring — load `skill-best-practices` instead.
- Do not apply wiki authoring structure to `SKILL.md` files (no ToC, no `<a id>` anchors in skills).
- Do not expand scope to writing style — load `writing-style-best-practices` for prose review.

## References

- [doc-content](references/doc-content.md) —
  full reference (load on demand)
- skill-authoring
  — for authoring skill files