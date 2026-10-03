---
source-slug: obsidian-cli
source-hash: 4865dd9b9d972d0c7dba3b6170a75c319c205479d05a82d7ee19108874c6101d
bundled: 2026-10-02
title: Obsidian CLI and Headless
type: concept
tags:
- obsidian
- cli
- automation
updated: 2026-10-02
---

# Obsidian CLI and Headless

Two official command surfaces — newer than the URI scheme and strictly
more capable for automation:

- **Obsidian CLI** (`obsidian`) drives the *running desktop app*.
- **Obsidian Headless** (`ob`, open beta) is a standalone client for
  Obsidian *services* — vault sync with no desktop app at all.

## Obsidian CLI

> "Anything you can do in Obsidian you can do from the command line."

Requires the Obsidian 1.12 installer; enable under Settings → General →
**Command line interface**. Runs individual commands or a TUI
(`obsidian` bare). If the app isn't running, the first command launches
it. If you only need sync without the app, that's Headless.

### Command surface

| Group | Commands |
| --- | --- |
| Navigation | `open`, `daily`, `search query=`, `read`, `tasks` |
| Write | `create name= content=`, `append`, `prepend`, `move`, `rename`, `delete`, `daily:append` |
| Inspection | `files`, `folders`, `file`, `links`, `backlinks`, `unresolved`, `tags counts`, `diff file= from= to=` |
| Commands | `commands` (list), `command id=` (run any palette command) |
| Vault | `vault=Name` prefixes any command to target a specific vault |
| Developer | `eval code="app.vault.getFiles().length"`, `plugin:reload id=`, `devtools`, `dev:screenshot` |

`obsidian eval` is the escape hatch — arbitrary JS against the live
`App`/`Plugin` objects, i.e. the entire plugin API
from a shell. `command` invokes *any* registered command, including ones
community plugins add — every plugin's command surface is automatically
CLI-addressable.

`unresolved` lists dangling wikilinks — a GUI-side check that parallels
the `wikilink-resolves` validator finding.

### vs the URI scheme

[obsidian-uri-scheme](./obsidian-uri-scheme.md) remains the zero-setup path (any URL emitter can
open a note, works everywhere). The CLI wins for scripts/agents: it can
*read* the vault, run searches, execute plugin commands, and return
structured output — things `obsidian://` can never do. Use the URI to
focus a page for the human; use the CLI to drive Obsidian as a tool.

## Obsidian Headless

`npm install -g obsidian-headless`, `ob login` (email/password/MFA).
Open-beta standalone client for Obsidian Sync: pull/push vaults with
end-to-end encryption, no desktop app. Obsidian's own positioning
includes "give agentic tools access to a vault without access to your
full computer" — shared vault → server/CI/agent reads it without the
user's session.

Distinction that matters: **CLI = control the app, Headless = sync the
data.** Headless cannot run plugin commands or open panes; the CLI
cannot sync without the app.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
