---
name: paniolo-obsidian-open
description: |
  Open a Markdown file or wiki page in the user's running Obsidian app via
  obsidian:// URIs. Use when handing a document to the user's GUI reader,
  opening a note at a filesystem path or inside a named vault, or making
  wikilink-navigable docs visible to the human. Do not use for creating
  or editing Paniolo wiki pages (use `paniolo wiki new`), for terminal
  rendering (use a Markdown renderer), or in headless sessions with no GUI.
license: MIT
metadata:
  version: 0.1.0
tags:
- obsidian
user-invocable: true
references:
- references/obsidian-cli.md
- references/obsidian-uri-scheme.md
- references/obsidian-vaults.md
- references/obsidian-wikilinks.md
---

# Obsidian Open

Open a Markdown file in the user's already-running Obsidian instance using
the `obsidian://` URI scheme — the bridge from terminal/agent work to the
user's GUI reader. The URI focuses the existing window; it never spawns a
duplicate app.

## Use When

- Presenting a doc or wiki page to the human — "open this in Obsidian".
- The user wants rendered Markdown with clickable `[[wikilinks]]`, graph
  view, or backlinks rather than terminal output.
- Handing off after agent work: point the human at the page you just wrote.

## Do Not Use

- Creating or editing Paniolo wiki pages — use `paniolo wiki new`; edits in
  Obsidian bypass frontmatter, prefix, and `wiki/log.md` conventions.
- Terminal rendering — use a Markdown renderer (glow or equivalent) instead.
- Headless or remote sessions with no desktop — the URI opens on the user's
  GUI session; if there is none, it does nothing useful.

## Recipe

1. **Prefer the Obsidian CLI when present.** Obsidian 1.12+ ships
   `obsidian` — it can open, read, and search, not just launch:

   ```text
   obsidian vault=<name> open path=<vault-relative-path>
   obsidian open file=<basename>            # resolve by name
   obsidian read file=<note>                # read without opening
   ```

   Probe with `obsidian version`; if it errors, fall through to the URI.
   Beyond opening, the CLI exposes `search`, `unresolved` (dangling
   wikilinks), `command id=` for any palette command, and `eval` for the
   full plugin API — prefer it whenever the goal is Obsidian *data*, not
   just focusing a page for the human.
2. **Fallback: locate the vault** for a URI launch. The vault is the
   folder registered in Obsidian containing the target file; vault name =
   folder basename. The registry lives in the app's global config
   (`%APPDATA%\Obsidian\obsidian.json` on Windows). If no registered vault
   contains the file, tell the user to open the folder as a vault once.
3. **Build the URI**, percent-encoding every value:

   ```text
   obsidian://open?vault=<name-or-id>&file=<vault-relative-path>
   obsidian://open?path=<absolute-path-percent-encoded>
   ```

   Prefer `path=` when the vault name is unknown — Obsidian finds the most
   specific registered vault containing that absolute path.
4. **Launch through the OS handler** (per-shell commands and quoting gotchas
   are in the URI-scheme reference):

   | Shell | Command |
   | --- | --- |
   | PowerShell | `Start-Process 'obsidian://…'` |
   | cmd | `start "" "obsidian://…"` |
   | macOS | `open 'obsidian://…'` |
   | Linux | `xdg-open 'obsidian://…'` |

## Output links — render file references as URIs

When the user's vault layout is known, render every Markdown-file reference
in your output as a clickable `obsidian://` link instead of a bare path:

```text
[design-paniolo-agent-browser.md](obsidian://open?vault=<vault>&file=<vault-relative%2Fpath%2Ffile.md>)
```

- Percent-encode the `file` value (`/` → `%2F`, spaces → `%20`).
- Only link files that live inside a **registered** vault — bare paths are
  better than links that silently do nothing.
- Markdown-file mentions only; don't wrap code symbols, directories, or
  non-doc files.
- Clicking a custom-scheme link in a terminal goes through the OS trust
  prompt ("unsafe location") on every click. When you can act, launch the
  URI yourself instead of asking the user to click. The click-side fix is
  the `paniolo-ai/herdr-obsidian` herdr plugin, pending upstream routing
  support for non-http schemes (herdrdev/herdr#4874).

## Verify

Tell the user what you opened and on which vault. A correct `open` focuses
Obsidian on the page; an unregistered vault or bad encoding silently does
nothing — when in doubt, test `obsidian://open?vault=<name>` alone first.

## References

- [obsidian-cli](references/obsidian-cli.md) — the preferred CLI surface:
  `open`/`read`/`search`/`command`/`eval`, vault targeting, and headless sync
- [obsidian-uri-scheme](references/obsidian-uri-scheme.md) — full action
  set, parameters, encoding, and the `x-callback-url` round-trip
- [obsidian-vaults](references/obsidian-vaults.md) — what a vault is,
  registration, and repo layouts
- [obsidian-wikilinks](references/obsidian-wikilinks.md) — link syntax and
  resolution inside the vault

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
