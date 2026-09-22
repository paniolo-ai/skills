# paniolo-ai/skills

Distribution registry for every Paniolo skill — the best-practice catalog plus the
product skills (`paniolo-scan`, `paniolo-scan-remediate`, `paniolo-config-init`,
`paniolo-config-upgrade`). It is also the **Claude Code plugin marketplace**:
`/plugin marketplace add paniolo-ai/skills` ships the whole catalog plus the
`/paniolo-*` slash commands (see [.claude-plugin/](.claude-plugin/)).

[![Install with skills.sh](https://skills.sh/b/paniolo-ai/skills)](https://skills.sh/paniolo-ai/skills)

## `skills/` is generated — do not edit it

Every `SKILL.md` and every file under `references/` is compiled output. The
source of truth is one page per skill in `sharp-shooter-wiki/wiki/skills/`,
named `paniolo-<name>.md`; `paniolo skills bundle` compiles those pages into the
directories here.

Edit the wiki page and re-bundle. A hand edit here survives only until the next
bundle, and until then it is invisible: the page it came from never learns about
it, so the change is silently reverted rather than reported.

`references/` files are compiled from the flat wiki pages named in each page's
`references:` list, which is why they carry `source-wiki` / `source-slug` /
`source-hash` provenance in their frontmatter.

Everything *outside* `skills/` is hand-authored: `.claude-plugin/` (the plugin and
marketplace manifests), `.agents/commands/` and `.agents/workflows/` (the `/paniolo-*`
slash commands and Antigravity workflows), and `.agents/agents/`.

## Layout

This repo follows the [skills.sh](https://skills.sh) multi-skill layout:

```text
skills/
└── paniolo-<name>/
    ├── SKILL.md
    └── references/   # bundled, self-contained reference pages
```

## Install

```bash
npx skills add paniolo-ai/skills --skill paniolo-typescript-best-practices
```

Or via the Paniolo harness CLI:

```bash
pnpm run skills -- add paniolo-ai/skills --skill paniolo-typescript-best-practices
```

Skills are copied into `.agents/skills/external/paniolo-<name>/` and tracked in
`skills-lock.json` for reconcile-aware updates.

## Versioning

Each `SKILL.md` declares `metadata.version` and `license`. Update via
`paniolo skills update <name>` (or `npx skills update <name>`); the lockfile
records the vendored-base hash so customizations are three-way merged, never
clobbered.
