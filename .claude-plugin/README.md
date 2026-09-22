# .claude-plugin

This directory packages the Paniolo skills registry as a **Claude Code plugin** — one
install ships the full skill catalog plus the `/paniolo-*` slash commands that turn the
diagnostic CLI into a "scan and fix" experience inside an agent session.

## Contents

| Path               | Role                                                                         |
| ------------------ | ---------------------------------------------------------------------------- |
| `plugin.json`      | Plugin manifest: name, version, description, author, license, homepage, repository, keywords, and the `skills`/`commands`/`agents` path fields below. |
| `marketplace.json` | Marketplace manifest so this repo is installable as a plugin (`source: ./`). |

The plugin's `skills` field points at [`./skills/`](/skills/) — the bundled registry
itself, so the plugin ships every published skill with no vendored copies. Slash
commands and subagent modes live under `.agents/` and are repointed via the manifest's
[component path fields](https://code.claude.com/docs/en/plugins-reference#component-path-fields):

```json
"skills": ["./skills/"],
"commands": ["./.agents/commands/"],
"agents": ["./.agents/agents/"]
```

Because this plugin's marketplace `source` resolves to the marketplace root (`"./"`), the
`skills` field *replaces* the default `skills/` scan rather than adding to it, so there is no
duplicate skill tree. `commands` and `agents` always replace their defaults. The
`/paniolo-scan`, `/paniolo-scan-remediate`, `/paniolo-config-init`, and
`/paniolo-config-upgrade` slash commands live in
[`.agents/commands/`](/.agents/commands/); subagent modes live in [`.agents/agents/`](/.agents/agents/).

## Installing and registering the command

```bash
# Register this repo as a marketplace, then install
/plugin marketplace add paniolo-ai/skills
/plugin install paniolo-scan@paniolo-ai
```

After installing, `/paniolo-scan` and the rest of the `/paniolo-*` commands appear in the
slash-command list, and every skill in the catalog is available for auto-invocation.
