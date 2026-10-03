---
source-slug: obsidian-uri-scheme
source-hash: 0b29fb21391cd0337d1d9e89e09e108da5423df83421ff180917b396196d508e
bundled: 2026-10-02
title: Obsidian URI Scheme
type: concept
tags:
- obsidian
- harness-eng
updated: 2026-10-02
status: draft
---

# Obsidian URI Scheme

`obsidian://` is a custom URI protocol registered by the Obsidian desktop app.
Anything that can open a URL — a shell, a terminal multiplexer, an agent, a
browser link — can drive the user's Obsidian window without Obsidian needing
an API server. This is the canonical "agent → GUI reader" bridge: open a
specific note in the user's already-running instance.

## URI format

```text
obsidian://<action>?param1=value&param2=value
```

Values must be percent-encoded (`/` → `%2F`, space → `%20`, `#` → `%23`) or
the URI parses incorrectly.

## Actions

| Action | What it does |
| --- | --- |
| `open` | Open a vault, or a file within a vault |
| `new` | Create a note, optionally with `content` |
| `daily` | Create or open today's daily note (needs Daily notes plugin) |
| `unique` | Create a uniquely-named note (needs Unique note creator plugin) |
| `search` | Open search, optionally with `query` |
| `choose-vault` | Open the vault manager |
| `hook-get-address` | Return/copy the `obsidian://` URI of the focused note |

## The `open` action — the agent primitive

```text
obsidian://open?vault=<name-or-id>&file=<vault-relative-path>
```

- `vault` — vault **name** (folder basename) or **vault ID** (16-char code,
  findable via vault switcher → "Copy vault ID").
- `file` — note name or path from vault root; `.md` extension optional.
- `path` — **absolute filesystem path**, overriding `vault`+`file`. Obsidian
  finds the most specific vault containing that path and opens the file.
  This is the agent-friendly form: no need to know the vault name.
- `paneType` — `tab` / `split` / `window`; default replaces the last active
  tab.
- `prepend` / `append` — write to the note while opening (merges
  properties).

Heading and block navigation work via encoding: `file=Note%23Heading` opens
a heading, `Note%23%5EBlock` a block.

Shorthand forms: `obsidian://vault/<vault>/<note>` equals
`open?vault=…&file=…`; `obsidian:///<abs/path>` equals `open?path=…`.

## Opening from a shell or agent

Opening the URI is idempotent — Obsidian focuses the existing window rather
than spawning a duplicate.

| Shell | Command |
| --- | --- |
| PowerShell | `Start-Process 'obsidian://open?vault=X&file=Y'` |
| Windows `cmd` | `start "" "obsidian://open?vault=X&file=Y"` |
| macOS | `open 'obsidian://open?vault=X&file=Y'` |
| Linux | `xdg-open 'obsidian://open?vault=X&file=Y'` |

PowerShell gotcha: quote the URI — an unquoted `&` is parsed as a command
separator. Windows `cmd` needs the empty-title argument `""` after `start`.

## Round-tripping: `x-callback-url`

`new`, `unique`, and `hook-get-address` accept `x-success` / `x-error`
callback parameters. When `x-success` is supplied, Obsidian calls it back
with the note's `name`, its `obsidian://` URI, and its `file://` URL — the
reverse direction of the bridge: user picks a note, agent learns its
address. Without a callback, `hook-get-address` copies the focused note's
`obsidian://open` URL to the clipboard.

## Registration and prerequisites

- The `obsidian://` protocol is registered when the app runs once on
  Windows/macOS; on Linux it needs a `.desktop` file with `Exec=… %u`.
- The target vault must already be **registered** in Obsidian (opened once
  through the vault picker, or present in `%APPDATA%\Obsidian\obsidian.json`
  on Windows). `open?vault=<name>` cannot find a vault Obsidian has never
  seen — but `open?path=<abs>` can locate any registered vault by path.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
