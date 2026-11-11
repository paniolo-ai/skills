---
name: paniolo-doc
description: |
  Documentation standards for Markdown documents: file naming, heading hierarchy, ToC, explicit anchor tags, doc placement, and QMD semantic retrieval formatting. Use when creating, restructuring, or auditing wiki pages, READMEs, or CONTRIBUTING files. Do NOT use for general prose writing style (load paniolo-writing) or in-code JSDoc comments.
license: MIT
metadata:
  version: 0.2.0
tags:
- documentation
- markdown
- structure
user-invocable: true
references:
- references/doc-content.md
- references/doc-structure.md
- references/markdown-anchor-links.md
- references/markdown-file-naming.md
- references/markdown-required-elements.md
---

**Requires:** file-read. No terminal needed unless validating wiki integrity or markdown lint.

# Documentation Standards

## Preconditions

- Read the target document before editing or restructuring.
- Determine whether the target is a wiki document, a project root document (`README.md`, `CONTRIBUTING.md`), or an agent skill (`SKILL.md`).
- For writing style, tone, active voice, or conciseness, load [paniolo-writing](../paniolo-writing/SKILL.md).
- For in-code comments, load the code comment skill.

## Doc Placement Decision Tree

```text
├── General project info (setup, onboarding)   → Root README.md or CONTRIBUTING.md
├── Architectural concepts & patterns          → <wiki-root>/<category>/
├── Feature or domain deep dive                → <wiki-root>/
├── Code-level implementation details          → Inline JSDoc / TSDoc comments
└── Skill definitions for AI agents            → /.agents/skills/<name>/SKILL.md
```

## Structural Standards

Every **wiki authoring page** (`<wiki-root>/*.md`) must follow:

1. **File naming**: lowercase kebab-case (`cache-invalidation.md`), no underscores, dots, or PascalCase.
2. **H1 title**: exactly one per document, at the very top.
3. **Brief intro**: 1–3 sentences immediately under H1 stating what the document covers and who should read it.
4. **Table of Contents**: required for documents >150 lines or with 3+ sections.
5. **Explicit `<a id="...">` anchors**: required for every heading listed in the ToC (wiki docs only; never in `SKILL.md`).
6. **ATX headers**: use `#`, `##`, `###` with one blank line before and after; never skip heading levels (e.g. `#` directly to `###`).
7. **Code blocks**: always fenced with explicit language specifiers (` ```typescript `, ` ```bash `).
8. **Section separators**: use `---` between major H2 sections only.
9. **`## See Also`**: at the bottom, linking related docs via wikilinks `[[slug]]` or relative markdown links.

## Retrieval Optimization (QMD)

Docs are indexed for semantic retrieval by agents and developers:

- **Keyword-rich H1**: The H1 embeds with every chunk; include the core domain term in the title.
- **Section granularity**: Keep sections ≤900 tokens (QMD chunk size). Put direct answers immediately under their headings.
- **Query aliases and exact terms**: Include the real commands, aliases, and filenames searchers query, not only abstract descriptions.
- **Focused code blocks**: Keep code snippets tight and colocated with the relevant explanation.

## Validation Checklist

- [ ] Filename is kebab-case with `.md` extension.
- [ ] Exactly one H1 title at the top.
- [ ] 1–3 sentence intro under H1.
- [ ] ToC present if >150 lines or 3+ sections.
- [ ] Explicit `<a id="...">` anchors match all ToC entries (wiki docs only).
- [ ] ATX headers in Title Case without skipped levels.
- [ ] Code blocks fenced with explicit language specifiers.
- [ ] `## See Also` at bottom with valid links.

## Evaluations (I/O Examples)

**Input:** "Create a new wiki page for our background queue worker"
**Expected:** Agent creates kebab-case file with H1, concise intro, structured sections, explicit anchors, and See Also links.

**Input:** "Audit this wiki page against documentation standards"
**Expected:** Agent verifies file naming, single H1, ToC presence, anchor tag matches, ATX header hierarchy, and fenced code languages.

## Do Not

- Do not use for general prose style, active voice, or PR drafting — load [paniolo-writing](../paniolo-writing/SKILL.md).
- Do not apply wiki structural elements (ToC, `<a id>` anchors) to `SKILL.md` files.
- Do not create orphan wiki pages; always link them from an index or related page.
- Do not skip heading levels (e.g. jumping from `#` to `###`).

## References

- [doc-content](references/doc-content.md)
- [doc-structure](references/doc-structure.md)
- [markdown-anchor-links](references/markdown-anchor-links.md)
- [markdown-file-naming](references/markdown-file-naming.md)
- [markdown-required-elements](references/markdown-required-elements.md)
- [paniolo-writing](../paniolo-writing/SKILL.md)

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
