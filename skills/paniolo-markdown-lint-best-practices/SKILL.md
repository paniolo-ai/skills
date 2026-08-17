---
name: paniolo-markdown-lint-best-practices
description: |
  Pre-authoring checklist and validation workflow for writing lint-clean markdown. Use when creating or editing any .md file to pass paniolo scan and paniolo wiki checks on the first try.
license: MIT
metadata:
  version: 0.1.1
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

- Creating or editing any `.md` file in the repository.
- Writing skills, agents, docs, or instruction files.
- Choosing the repository's current Markdown, wiki, and lint entry points without
  relying on retired script aliases.

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

- Internal harness links use workspace-relative paths (`<repo>/<path>`,
  `/.agents/skills/...`), not `./file.md`. References to a wiki page are the
  exception — use cross-wiki wikilink syntax (`[[wiki-name:slug]]`) rather than a
  path, so `paniolo wiki rename`/`move` can rewrite them.
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

Use the command that owns the edited file. Inspect the repository's current
scripts before running a dedicated Markdown command; `lint:md` is not a
portable or guaranteed entry point.

| Scope | Command | What it checks |
| --- | --- | --- |
| Paniolo wiki page | `paniolo wiki` from the harness | Whole-wiki schema, sources, links, indexes, and orphans |
| Harness Markdown or prose not covered by the wiki gate | the project's lint script | Repository lint entry point, including `paniolo scan` guidance checks |

For a Paniolo wiki edit, run the whole-wiki gate first. If it does not cover
the edited page's prose checks, run the edited repository's lint entry point.
For the harness, the current sequence is:

```bash
paniolo wiki
<your lint command>
```

Do not treat a missing script as a successful validation step. Record it as
not run, select a current repository entry point, and execute that command.

---

## Prompt Template

```text
Read the repository's Markdown lint skill before writing any .md file.
Apply the pre-authoring checklist: structure, anchors (wiki authoring only), links, and line limits.
After writing a Paniolo wiki page, run paniolo wiki from the harness.
If that gate does not cover the edited prose, run the edited repository's current lint entry point.
Do not assume a lint:md script exists; inspect the repository scripts first.
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
- Do not skip `paniolo wiki` when editing wiki pages.
- Do not exceed the file size limits for skills and instruction files.
