---
source-slug: obsidian-vaults
source-hash: 3df3a8f28b265f3e18fc57e045b44d6c28be047c5d453732247c72fcdd688278
bundled: 2026-10-02
title: Obsidian Vaults
type: concept
tags:
- obsidian
- harness-eng
updated: 2026-10-02
status: draft
---

# Obsidian Vaults

A **vault** is just a folder on the local filesystem — a root directory plus
every subfolder beneath it. Obsidian stores notes as plain Markdown files
inside it, watches the folder for external changes, and keeps up as other
editors and tools write to the same files. There is no import step and no
proprietary database: the folder *is* the data.

## Anatomy

- **Notes and folders** — ordinary `.md` files and subfolders; edit them
  with any tool and Obsidian refreshes to match.
- **`.obsidian/`** — per-vault config directory at the vault root (hotkeys,
  themes, plugins, workspace layout). `workspace.json` changes constantly
  and belongs in `.gitignore` when a vault is version-controlled.
- **Global registry** — vaults are registered app-wide, not self-declaring.
  On Windows the registry is `%APPDATA%\Obsidian\obsidian.json` mapping
  vault IDs (16-char codes) to folder paths. A folder becomes "a vault"
  only once registered — opened once through the vault picker or pre-seeded
  in that file.
- **Vault name** — the folder's basename; it is what `obsidian://` URIs and
  the vault switcher display.

## Rules that matter for automation

- **Links are vault-local** — `[[x]]` never resolves into another vault,
  and Obsidian warns against vaults nested inside vaults (link updates get
  confused). The vault root choice defines the link graph's reach.
- **One folder, many tools** — because storage is plain files, an agent, a
  git checkout, and the Obsidian app can share one directory safely.
  Obsidian is a reader/writer, not an owner.
- **Multiple vaults** can coexist (work vs. personal); each is a separate
  link universe.

## Vault-over-repos: the Paniolo pattern

A vault opened at a directory *above* multiple `wiki/` roots sees every
corpus's pages as folders in one vault — bare `[[slug]]` links then resolve
across corpora, sidestepping the per-vault link boundary. The trade-offs
(this session's working setup uses exactly this):

- **Slug uniqueness becomes load-bearing** — same-named pages in different
  corpora (`index.md`, `log.md`, `authoring.md`) resolve ambiguously.
- **Non-wiki directories get indexed too** — `raw/` snapshots and other
  repos inflate search results unless excluded in vault settings.
- **`.obsidian/` stays out of the corpora** — config lives at the vault
  root, so nothing Obsidian-owned pollutes the wiki repos.

An alternative that keeps each corpus clean: a container folder holding
junctions (`mklink /J`) into each `wiki/` directory — the vault sees two
folders of pages and nothing else.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
