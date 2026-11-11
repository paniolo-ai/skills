---
name: paniolo-doc
description: |
  Documentation and writing standards for Markdown docs: file naming, structure, ToC, anchors, qmd retrieval, clarity, active voice, audience, tone, and conciseness. Use when authoring, editing, or reviewing wiki pages, READMEs, or CONTRIBUTING files. Do NOT use for in-code JSDoc comments or SKILL.md files.
license: MIT
metadata:
  version: 0.2.0
tags:
- documentation
- writing
user-invocable: true
references:
- references/active-vs-passive-voice.md
- references/audience-and-tone.md
- references/avoid-jargon-unless-necessary.md
- references/doc-content.md
- references/doc-structure.md
- references/markdown-anchor-links.md
- references/markdown-file-naming.md
- references/markdown-required-elements.md
- references/writing-style-clarity-and-conciseness.md
- references/writing-style-writing-style.md
---

**Requires:** file-read. No terminal needed unless validating markdown lint or wiki integrity.

# Documentation and Writing Standards

## Preconditions

- Read the target document before editing or reviewing.
- Determine whether the target is a wiki authoring doc, a project root doc (README/CONTRIBUTING), or a skill definition.
- For in-code comments, load the code comment skill instead.

## Doc Placement Decision Tree

```text
├── General project info (setup, contributing) → Root README.md or CONTRIBUTING.md
├── Coding standards & architectural concepts  → <wiki-root>/<category>/
├── Feature or domain deep dive                → <wiki-root>/
├── Code-level implementation details          → Inline JSDoc / TSDoc comments
└── Skill definitions for AI agents            → /.agents/skills/<name>/SKILL.md
```

## Structural Standards

Every **wiki authoring page** (`<wiki-root>/*.md`) must follow:

1. **File naming**: lowercase kebab-case (`doc-conventions.md`), no underscores or PascalCase.
2. **H1 title**: exactly one per document, at the very top.
3. **Brief intro**: 1–3 sentences immediately under H1 stating what this is and who should read it.
4. **Table of Contents**: required for documents >150 lines or with 3+ sections.
5. **Explicit `<a id="...">` anchors**: required for every heading listed in the ToC (wiki docs only; never in `SKILL.md`).
6. **ATX headers**: use `#`, `##`, `###` with one blank line before and after; never skip heading levels.
7. **Code blocks**: always fenced with language specifiers (` ```typescript `, ` ```bash `).
8. **Section separators**: use `---` between major H2 sections only.
9. **`## See Also`**: at the bottom, linking related docs via wikilinks `[[slug]]` or relative markdown links.

## Writing Style Standards

### Clarity and Conciseness

- Write for comprehension, not token padding.
- Keep sentences short (<25 words preferred).
- Break complex ideas into bulleted or numbered lists.
- Define acronyms on first use: `"AST (Abstract Syntax Tree)"`.

### Audience and Tone

- Target: competent engineer new to the project. Skip basic language fundamentals they already know.
- Avoid condescending filler: `"Obviously..."`, `"Simply..."`, `"Just..."`.
- State rules directly and explain the "why" when constraints appear arbitrary.

### Active Voice

- Prefer active voice for directness and clarity.
- Use passive voice only when the actor is genuinely unknown or irrelevant.
- *Active:* "The parser throws an error when the token is missing."
- *Avoid:* "An error is thrown by the parser when the token is missing."

### Terminology and Jargon

- Define technical or project-specific terms on first introduction.
- Prefer plain language until the term has been defined.

## Retrieval Optimization (QMD)

Docs are indexed for semantic retrieval by agents and developers:

- Use keyword-rich, specific H1 titles (the title embeds with every chunk).
- Structure sections into focused units (≤900 tokens). Put direct answers immediately under their headings.
- Include real terms, commands, and aliases searchers actually query, not only abstract descriptions.
- Keep code snippets tight and relevant.

## Validation Checklist

- [ ] Filename is kebab-case with `.md` extension.
- [ ] Exactly one H1 title at the top.
- [ ] 1–3 sentence intro under H1.
- [ ] ToC present if >150 lines or 3+ sections.
- [ ] Explicit `<a id="...">` anchors match all ToC entries (wiki docs only).
- [ ] ATX headers in Title Case without skipped levels.
- [ ] Code blocks fenced with explicit language specifiers.
- [ ] Active voice and sentences <25 words.
- [ ] No condescending words ("simply", "obviously").
- [ ] `## See Also` at bottom with valid links.

## Evaluations (I/O Examples)

**Input:** "Review this doc section for style and structure"
**Expected:** Agent outputs a structured Passes/Gaps report citing specific lines, checking both structural rules (headings, anchors) and writing style (active voice, clarity, jargon).

**Input:** "Create a new wiki page explaining our cache invalidation model"
**Expected:** Agent creates kebab-case file with H1, concise intro, ToC with explicit anchors if long, clear active voice, and See Also links.

## Do Not

- Do not use for in-code JSDoc comments — load the code comment skill.
- Do not apply wiki structural elements (ToC, `<a id>` anchors) to `SKILL.md` files.
- Do not fabricate facts or citations.
- Do not write verbose, passive, or condescending prose.

## References

- [doc-content](references/doc-content.md)
- [doc-structure](references/doc-structure.md)
- [markdown-anchor-links](references/markdown-anchor-links.md)
- [markdown-file-naming](references/markdown-file-naming.md)
- [markdown-required-elements](references/markdown-required-elements.md)
- [writing-style-writing-style](references/writing-style-writing-style.md)

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
