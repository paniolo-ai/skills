---
name: paniolo-markdown-lint-best-practices
description: |
  Pre-authoring checklist and validation workflow for writing lint-clean markdown in this repo. Use when creating or editing any .md file to pass paniolo-scan and wiki-io checks on the first try.
license: MIT
metadata:
  version: 0.1.0
tags:
- markdown
- lint
references:
- 'wiki: sharp-shooter-wiki:doc-content → references/doc-content.md'
- 'wiki: sharp-shooter-wiki:markdown-lint → references/markdown-lint.md'
---

**Requires:** file-read, terminal (markdown linting).

**Full reference:** [markdown-lint](references/markdown-lint.md)

**Paniolo repo:** see
scan-markdown-lint

# Markdown Lint Best Practices

## Use When

- Creating or editing any `.md` file in this repo.
- Writing skills, agents, docs, or instruction files.
- Trying to pass `pnpm run lint:md` and `pnpm run check:wiki` without repeated lint cycles.

---

## Pre-Authoring Checklist

Apply these before writing a single line — each prevents a common first-pass failure.

### Structure

- Blank line before AND after every heading — no exceptions.
- Headings increment one level at a time (no `##` jumping to `####`).
- Headings use Title Case; no trailing punctuation (no periods, no colons).
- Every fenced code block declares a language (` ```typescript `, ` ```bash `, ` ```json `).
- Prose lines ≤ 100 characters — headings, tables, and code blocks are exempt.
- Unordered lists use `-`; ordered lists use `1.` for every item.
- No bare URLs — always `[text](url)`.
- Do not use `**Bold Text**` as a heading substitute — use `##` instead.

### Links

- Internal harness links use workspace-relative paths (`sharp-shooter-wiki/wiki/...`,
  `/.agents/skills/...`), not `./file.md`.
- Fragment links (`#anchor`) must resolve to a real heading or `<a id="...">` in the **target**
  doc file (not in skills).
- No `file://` links.

### File size limits

- `.agents/skills/*/SKILL.md` and `agents/**/*.md`: ≤ 159 lines.
- rules: ≤ 100 lines; plain headings only
  (no anchors or ToC).
- `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, adapter files: ≤ 150 lines. Keep `AGENTS.md`
  brief; full skill slug list: available-skills.

---

## Validation Stages

Run the narrowest useful check first, then the wiki corpus check when a wiki
page changed.

| Step | Command               | What it checks                                      |
| ---- | --------------------- | --------------------------------------------------- |
| 1    | `pnpm run lint:md`    | `paniolo-scan --fail-on error` for harness guidance |
| 2    | `pnpm run check:wiki` | Wiki schema, sources, links, indexes, and orphans   |

Full suite:

```bash
pnpm run lint:md && pnpm run check:wiki
```

---

## Prompt Template

```text
Read .agents/skills/markdown-lint-best-practices/SKILL.md before writing any .md file.
Apply the pre-authoring checklist: structure, anchors (wiki authoring only), links, and line limits.
After writing, run pnpm run lint:md and, for wiki pages, pnpm run check:wiki.
Do not finish with red validation.
```

---

## References

- **Full reference:** [markdown-lint](references/markdown-lint.md)
- **Doc standards:** [doc-content](references/doc-content.md)
- **Skill standards:** [skill-best-practices](../paniolo-skill-best-practices/SKILL.md)
- **Config:** `paniolo.config.json`

## Do Not

- Do not add `<a id="..."></a>` anchors in `.agents/skills/*/SKILL.md` or
  `.agents/rules.md`.
- Do not omit blank lines around headings.
- Do not use bare URLs in prose — always `[text](url)`.
- Do not skip `pnpm run check:wiki` when editing wiki pages.
- Do not exceed the file size limits for skills and instruction files.