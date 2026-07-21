# paniolo-ai/skills

Source registry for Paniolo-vendored authoring skills.

[![Install with skills.sh](https://skills.sh/b/paniolo-ai/skills)](https://skills.sh/paniolo-ai/skills)

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
