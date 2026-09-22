# .agents

Harness-specific surfaces this repo ships alongside the generated skills catalog:
slash commands, agent modes, and workflows.

## Contents

- [`commands/`](commands/) — Claude Code plugin slash commands (`/paniolo-scan`,
  `/paniolo-scan-remediate`, `/paniolo-config-init`, `/paniolo-config-upgrade`),
  repointed here via [`plugin.json`](/.claude-plugin/plugin.json)'s `commands` field.
- [`agents/`](agents/) — Claude Code plugin subagent modes (e.g.
  [`adapter-sync.agent.md`](agents/adapter-sync.agent.md)), repointed here via `plugin.json`'s
  `agents` field.
- [`workflows/`](workflows/) — Antigravity-specific slash-command workflows (see below).
  Antigravity has no equivalent manifest remapping, so its workflow files live under their own
  vendor-specific name rather than `commands/`.

The portable skills themselves are generated into [`/skills/`](/skills/) from
`sharp-shooter-wiki` source pages — see the top-level [README](/README.md). The plugin ships
them via `plugin.json`'s `skills` field; they are not authored here either.

## Antigravity workflows

Antigravity does **not** support native automated hooks (unlike Claude Code, Copilot, and
Cursor). It uses manual slash-command workflows instead.

- [`workflows/paniolo-scan.md`](workflows/paniolo-scan.md) — the `/paniolo-scan` slash command.
  Audit this repo's AI-agent harness with the deterministic paniolo-scan tool (read-only).
- [`workflows/paniolo-scan-remediate.md`](workflows/paniolo-scan-remediate.md) — the
  `/paniolo-scan-remediate` slash command. Fix the audit's findings in the working tree, then
  re-scan for the delta.
