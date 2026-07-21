---
name: paniolo-skill-best-practices
description: |
  How to write, update, split, or review SKILL.md files in this repo. Use when creating a new skill, updating an existing one, or auditing for quality and conventions.
license: MIT
metadata:
  version: 0.1.0
tags:
- skills
- authoring
references: []
user-invocable: true
---

**Requires:** file-read. No terminal or network access needed.

# Skill Best Practices

## Use When

- Creating a new `SKILL.md` under `.agents/skills/`.
- Updating an existing skill after learning new patterns.
- Deciding whether content belongs in a skill, a doc, or a workflow.
- Reviewing or auditing a skill for quality and convention compliance.

---

## Core Rules

1. **Skills must be ≤ 500 lines.** Enforced by `@paniolo/scan` (configurable via
   `paniolo.config.json` `skill.maxLines`).
2. **Skills are pointers, not encyclopedias.** Tell the agent _what to do_ and _where to look_.
   Defer detail to `sharp-shooter-wiki/wiki/` leaves.
3. **Docs are canonical.** Update the doc first; then update the skill to point at it.
4. **No secrets.** Never embed credentials or PII in a skill or examples.
5. **Naming:** harness first-party catalog skills use generic slugs (e.g.
   `code-comment-best-practices`) under `.agents/skills/<slug>/`. Paniolo-vendored
   skills live under `.agents/skills/external/paniolo-<name>/` and use `paniolo-<name>` as the
   slug. Repo-local skills live under `.agents/skills/repos/<repo>/<slug>/` —
   the path namespaces them, so slugs need not repeat the repo prefix. The `name` frontmatter must
   match the directory name. Sibling repos should not copy catalog skills — load the harness
   skill plus that repo's supplement under `sharp-shooter-wiki/wiki/`.

---

## What Goes Where

| Content type                              | Where it lives                             |
| ----------------------------------------- | ------------------------------------------ |
| Step-by-step task workflow                | `.agents/workflows/<name>.md`              |
| Full pattern reference with many examples | `sharp-shooter-wiki/wiki/`                 |
| Quick lookup table / rule summary         | wiki authoring leaf or repo `docs/`        |
| "Use this when X, do Y, see Z"            | `SKILL.md`                                 |
| Project background / motivation           | `sharp-shooter-wiki/wiki/` or repo `docs/` |

---

## Structure Template

Every `SKILL.md` must follow this structure. `## Use When` is optional — omit it
when the `description` frontmatter already carries complete trigger and exclusion language.
Add it only when trigger conditions are too complex to express in one sentence.

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
3. Update the skill with deep links (e.g. `sharp-shooter-wiki:<topic>` in prose).
4. Run `npx @paniolo/scan` to verify ≤ 500 lines.

**Output format for agents:**

- _Creating:_ Output the full file in one code block + brief decision list.
- _Reviewing:_ Output a **Passes** list and a **Gaps** table (item | issue | fix).
- _Updating:_ Output only changed sections + one sentence explanation per change.

---

## Deep Linking

Use cross-wiki wikilinks to reference wiki pages (for example
`sharp-shooter-wiki:<topic>`). Do not add `<a id="..."></a>` tags in `SKILL.md`.
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
`skill-link-integrity` separately checks that every link actually resolves. Full detail:
skill-composition.

---

## Maintenance & Review Checklist

- [ ] `name` and `description` are clear and conform to rules.
- [ ] `license`, `metadata.version`, and `tags` are present for skills shipped to customers.
- [ ] `user-invocable: true` if it should be a slash command.
- [ ] Body is ≤ 500 lines; references are one level deep.
- [ ] Output format is explicitly specified for the agent.
- [ ] Error handling (fast-fail) and clarifying-question policy defined.
- [ ] Related/dependent skills are linked by full path, not bare slug.
- [ ] No secrets or PII.

---

## References

- Full design guide: skill-authoring
- Skill composition: skill-composition
- Line-count check: `@paniolo/scan` (`skill-line-count` rule)
- Workflows: [`.agents/workflows/`](/.agents/workflows/)

## Do Not

- Do not put long code examples in a skill — put them in `docs/` and link.
- Do not exceed 500 lines — the CI check will fail.
- Do not duplicate content between a skill and a doc — the doc is canonical.
- Never ask the user whether to commit code changes or open a PR.