---
name: paniolo-skill-best-practices
description: |
  How to write, update, split, or review SKILL.md files. Use when creating a new skill, updating an existing one, or auditing for quality and conventions.
license: MIT
metadata:
  version: 0.1.1
tags:
- skills
- authoring
user-invocable: true
references: []
---

**Requires:** file-read, file-edit, terminal (Rust validation).

# Skill Best Practices

## Use When

- Creating a new `SKILL.md` under `.agents/skills/`.
- Updating an existing skill after learning new patterns.
- Deciding whether content belongs in a skill, a doc, or a workflow.
- Reviewing or auditing a skill for quality and convention compliance.

---

## Core Rules

1. **Skills must be ≤ 500 lines (100–300 lines optimal).** Enforced by `paniolo scan` (`skill-line-count` rule; configurable via `paniolo.config.json` `skill.maxLines`). SkillsBench v4 benchmark shows 2–3 focused skills (+19.0pp) outperform 4+ overlapping skills (+10.1pp), and compact workflows beat exhaustive manuals.
2. **3-Tier Progressive Disclosure:**
   - **Tier 1 (Catalog/Discovery):** Frontmatter `name` + `description`. Budget under 1,536 characters (Claude Code silently truncates beyond this; `skill-description-budget` warns at 80% / 1,228 chars).
   - **Tier 2 (Core Workflow):** `SKILL.md` body (≤500 lines). Entry point, required tools, step-by-step numbered flow, and fast-fail rules.
   - **Tier 3 (Deep Reference & Automation):** Bundled `references/` leaves and deterministic `scripts/`. Loaded or executed only on demand.
3. **Triggering & Negative Boundaries:** Description and `## Do Not` define clear negative boundaries ("Do NOT use for...") to stop over-triggering. Avoid pushy keywords ("always", "crucial", "essential") that flood the orchestrator.
4. **Disclose Side Effects & Require Confirmations:** If a skill writes files, commits, or mutates config, disclose in `description:` (`skill-side-effect-disclosure`). Any destructive action (delete, rm, drop, force, overwrite, truncate) must require a 2-step cue: candidate listing, `--dry-run`, or explicit confirmation (`skill-destructive-verb-confirmation`).
5. **Deterministic Scripts over Freehand Prose:** Move repetitive CLI or file-manipulation sequences into executable scripts under `scripts/` rather than multi-paragraph instructions.
6. **Docs are canonical.** Update the doc first; then update the skill to point at it.
7. **No secrets.** Never embed credentials or PII in a skill or examples.
8. **Naming:** harness first-party catalog skills use generic slugs (e.g. `code-comment-best-practices`) under `.agents/skills/<slug>/`. Paniolo-vendored skills live under `.agents/skills/external/paniolo-<name>/` and use `paniolo-<name>` as the slug. Repo-local skills live under `.agents/skills/repos/<repo>/<slug>/` — the path namespaces them, so slugs need not repeat the repo prefix. The `name` frontmatter must match the directory name. Sibling repos should not copy catalog skills — load the harness skill plus that repo's supplement in your wiki.

---

## What Goes Where

| Content type                              | Where it lives                       |
| ----------------------------------------- | ------------------------------------ |
| Step-by-step task workflow                | `.agents/workflows/<name>.md`        |
| Full pattern reference with many examples | a wiki page, bundled as `references/`|
| Quick lookup table / rule summary         | wiki authoring leaf or repo `docs/`  |
| "Use this when X, do Y, see Z"            | `SKILL.md`                           |
| Project background / motivation           | a wiki page or repo `docs/`          |

---

## Structure Template

Every `SKILL.md` must follow this structure. `## Use When` is optional — omit it when the `description` frontmatter already carries complete trigger and exclusion language. Add it only when trigger conditions are too complex to express in one sentence.

```markdown
---
name: <kebab-case-name>
description: <one sentence — what this covers and when to use it>
---

**Requires:** <tools>

# <Title>

## Use When <!-- optional: omit when description is sufficient -->

<bullet list of trigger conditions>

---

## Execution Workflow (if multi-step)

<numbered steps>

---

## Key Patterns

<only essential facts — defer detail to docs>

---

## References

<links to docs/, workflows, and related skills>

## Do Not

<short list of common mistakes to avoid>
```

For skills that ship to customers (Paniolo-vendored or registry skills), also
add `license`, `metadata.version`, and `tags` to the frontmatter:

```yaml
---
name: <kebab-case-name>
description: <one sentence — what this covers and when to use it>
license: MIT
metadata:
  version: 0.1.0
tags:
  - <technology-or-topic>
---
```

---

## Execution Workflow

**When creating or updating:**

1. Read the current skill and the doc it points to.
2. Add new patterns to the **doc** first.
3. For Paniolo's published catalog, edit the source page in
   `sharp-shooter-wiki/wiki/skills/`. The registry's `SKILL.md` and
   `references/` are generated by `paniolo skills bundle`; do not edit them
   directly. For other skills, update the authored `SKILL.md`.
4. Use `paniolo scan --fail-on warn` to validate skill frontmatter YAML,
   required fields, links, and size. The Rust `skill-frontmatter-yaml` rule
   parses YAML; Python and PyYAML are not required.
5. For a Paniolo catalog skill, run `paniolo wiki` on the source wiki and
   `paniolo skills bundle --check` against the generated registry.

**Output format for agents:**

- _Creating:_ Output the full file in one code block + brief decision list.
- _Reviewing:_ Output a **Passes** list and a **Gaps** table (item | issue | fix).
- _Updating:_ Output only changed sections + one sentence explanation per change.

---

## Deep Linking

Use cross-wiki wikilinks to reference wiki pages — `[[<wiki-name>:<topic>]]`, where
`<wiki-name>` is the name that wiki carries in `paniolo.config.json`. A wikilink is
what `paniolo wiki rename` and `paniolo wiki move` rewrite; a markdown path is not.
Do not add `<a id="..."></a>` tags in `SKILL.md`.
Use root-relative links only for non-wiki files (skills, agents, docs).

## Skill-to-Skill Linking

When a skill has a hard dependency on a companion skill (the task cannot be done correctly without
it), declare it near the top with an absolute link to a real skill, e.g. `**Depends on:**`
[`paniolo-typescript-best-practices/SKILL.md`](../paniolo-typescript-best-practices/SKILL.md). For
softer "see also" relationships, link the same way in `## References` — a bare slug mention with no
path is discoverable to an agent but not clickable for a human, so prefer the full link.
Avoid deep reference chains; prefer one cohesive skill over micro-skills unless each part is
genuinely reusable alone. `skill-link-style` (configurable via `linkStyle` in
`paniolo.config.json`) enforces this root-relative convention across all `SKILL.md` files;
`skill-link-integrity` separately checks that every link actually resolves.

---

## Maintenance & Review Checklist

- [ ] `name` and `description` are clear, concise, and conform to rules.
- [ ] `description` is under 1,536 characters (Claude Code cap; `skill-description-budget` warns at 80% / 1,228 chars).
- [ ] `description` includes explicit negative boundaries ("Do NOT use for...") and avoids pushy keywords.
- [ ] Side effects (file writes, git commits, configuration edits) are disclosed in `description:` (`skill-side-effect-disclosure`).
- [ ] Destructive actions (delete, rm, drop, force, overwrite, truncate) require a two-step confirmation or `--dry-run` (`skill-destructive-verb-confirmation`).
- [ ] Repetitive execution steps are packaged as deterministic scripts under `scripts/`.
- [ ] `license`, `metadata.version`, and `tags` are present for skills shipped to customers.
- [ ] `user-invocable: true` if it should be a slash command.
- [ ] Body is ≤ 500 lines (100–300 optimal); references are one level deep.
- [ ] Output format is explicitly specified for the agent.
- [ ] Error handling (fast-fail) and clarifying-question policy defined.
- [ ] Related/dependent skills are linked by full path, not bare slug.
- [ ] No secrets or PII.

---

## References

- Line-count check: `paniolo scan` (`skill-line-count` rule)
- Description budget check: `paniolo scan` (`skill-description-budget` rule)
- Negative boundaries check: `paniolo scan` (`skill-missing-do-not` rule)
- Side effect disclosure check: `paniolo scan` (`skill-side-effect-disclosure` rule)
- Destructive confirmation check: `paniolo scan` (`skill-destructive-verb-confirmation` rule)
- Link checks: `paniolo scan` (`skill-link-style`, `skill-link-integrity` rules)
- Workflows: [`.agents/workflows/`](/.agents/workflows/)

## Do Not

- Do not put long code examples in a skill — put them in `docs/` and link.
- Do not exceed 500 lines — the CI check will fail (`skill-line-count`).
- Do not exceed 1,536 characters in frontmatter description — Claude Code silently truncates it.
- Do not omit negative boundaries — always specify when NOT to use the skill.
- Do not perform unconfirmed destructive actions (deletes, overwrites, drops) without a dry-run or two-step cue.
- Do not duplicate content between a skill and a doc — the doc is canonical.
- For Paniolo work, do not add Python validation when the Rust CLI covers the check.
- Never ask the user whether to commit code changes or open a PR.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
