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
  version: 0.1.3
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

Open a Markdown file in the user's already-running Obsidian instance using the `obsidian://` URI scheme — the bridge from terminal/agent work to the user's GUI reader. The URI focuses the existing window; it never spawns a duplicate app.

## Use When

- Presenting a doc or wiki page to the human — "open this in Obsidian".
- The user wants rendered Markdown with clickable `[[wikilinks]]`, graph view, or backlinks rather than terminal output.
- Handing off after agent work: point the human at the page you just wrote.

## Do Not Use

- Creating or editing Paniolo wiki pages — use `paniolo wiki new`; edits in Obsidian bypass frontmatter, prefix, and `wiki/log.md` conventions.
- Terminal rendering — use a Markdown renderer (glow or equivalent) instead.
- Headless or remote sessions with no desktop — the URI opens on the user's GUI session; if there is none, it does nothing useful.

## Recipe

1. **Prefer the Obsidian CLI when present.** Obsidian 1.12+ ships `obsidian` — it can open, read, and search, not just launch:

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
   obsidian://open?vault=<name-or-id>&file=<vault-relative-path.md>
   obsidian://open?path=<absolute-path-percent-encoded>
   ```

   Prefer the vault form: take the registered vault whose path contains the
   file, use its folder basename (or id) as `vault`, and give `file` as the
   path *relative to that vault root*, with the `.md` extension. When a repo
   is checked out inside a larger vault (for example vault `meta` holding
   `paniolo/repos/<repo>/wiki/…`), the relative path is the long one.
   `path=<absolute>` is the fallback only: it relies on Obsidian resolving
   the most specific registered vault, and an absolute-path launch of a file
   in such a nested checkout failed to open while the vault form worked.
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

A URI launch gives no success signal of its own: `Start-Process` returns
cleanly even when Obsidian then reports it cannot open the file, and the
window title does not reliably name the note. Check the app's own state
instead. Obsidian records the open note in `<vault>/.obsidian/workspace.json`
within a second or two, so after the launch poll that file for the note's
name for up to about 10 seconds. A missing note never appears there, so the
check can fail as well as pass.

```powershell
$ws = "$vaultRoot\.obsidian\workspace.json"
1..10 | ForEach-Object {
  Start-Sleep 1
  if ((Get-Content $ws -Raw) -match [regex]::Escape($noteName)) { return 'opened' }
}
```

Report "opened" only when the note appears. If it does not, work through
these in order:

1. **The path or vault is wrong.** The URI uses the vault name or id and the
   vault-relative path with `.md`. "Vault not found" points at the name or id;
   "file not found" for a note that is on disk and was created long ago points
   at the path.
2. **Obsidian is hung or overloaded.** A healthy Obsidian opens a note
   created seconds earlier within about a second, in a plain folder, inside a
   git checkout, and with any filename. When a burst of thousands of file
   events hits the vault (a cargo build writing into a `.target` directory
   inside it, for example), Obsidian stops indexing new files. Measured
   2026-10-17 with a 40,000-file burst: from outside the app, touching the
   file, rewriting it, renaming it, opening by note name instead of path, and
   waiting over six minutes all failed. **Only restarting Obsidian fixed it**:
   the stuck notes opened within two seconds afterwards. Do not keep retrying;
   tell the user Obsidian looks hung and ask them to restart it, then retry.
3. Do not ask the user to "reload the vault" as the fix for this case, and do
   not write files into the vault to test it.

Prevent the second case instead of recovering from it: keep build and cache
directories out of the vault. The gate's scratch directories (`C:\lt-check-*`)
already are; a `.target` inside a repo that sits under the vault is not.

Keystrokes typed while Obsidian is hung or focused can land in the note you
just opened and edit it. This happened twice on 2026-10-17: a reply typed in
the terminal appeared as "yes" at the top of an open plan page. Open pages for
the user in **Reading view**, where typing does nothing.

The vault setting is Settings, Editor, "Default view for new tabs", set to
Reading view. Obsidian stores it in `<vault>/.obsidian/app.json` as
`"defaultViewMode": "preview"` (verified on 2026-10-17 by changing the setting
and reading the file). Before opening a page for the user, read that file; if
`defaultViewMode` is not `preview`, say so and suggest the setting. Do not
edit `app.json` yourself while Obsidian is running; the app rewrites it. The
setting applies to tabs opened afterwards: a note already open in an editor
tab keeps its mode until the user toggles it. There is no URI parameter for
the view mode (`paneType` only chooses tab, split, or window).

Even in Reading view, check the wiki's `git status` after opening pages for an
edit you did not make.

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
