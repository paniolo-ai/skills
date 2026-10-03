---
source-slug: obsidian-wikilinks
source-hash: 8e4e47b7b06faf8e6fc75a7efad4ba2d064c9adaae0438d3f96a21addc044e3a
bundled: 2026-10-02
title: Obsidian Wikilinks
type: concept
tags:
- obsidian
- harness-eng
updated: 2026-10-02
status: draft
---

# Obsidian Wikilinks

`[[wikilinks]]` are Obsidian's native link format — compact, rename-safe, and
resolved across the whole vault. They are why a folder of plain Markdown
becomes a navigable knowledge base instead of a pile of files.

## Syntax

| Form | Effect |
| --- | --- |
| `[[Note name]]` | Link to a note (`.md` extension optional) |
| `[[Note name\|Display]]` | Link with alias text |
| `[[Note#Heading]]` | Link to a heading inside the note |
| `[[Note#^block-id]]` | Link to a specific block |
| `[[Folder/Note]]` | Link by vault-relative folder path |
| `![[Note]]` | Embed the note's content inline |
| `[[Figure 1.png]]` | Non-Markdown files need the extension |

Markdown-format links `[text](Note%20name)` are also supported and equivalent;
Obsidian can be configured to prefer either. Folder paths use forward slashes
even on Windows.

## Resolution rules that matter for agents

- Links resolve **within the vault** — folder location is optional context,
  so a bare `[[slug]]` finds `wiki/slug.md` wherever it sits.
- Folder-qualified links (`[[a/b/note]]`) disambiguate same-named notes.
- Clicking a link to a **nonexistent** note creates the file — at the
  folder path if one is given, otherwise at the configured default location.
- These characters may not work inside link text: `# | ^ : %% [[ ]]`.
- Rename updates all links automatically (a setting can prompt instead).
- Links are **local to one vault**: `[[x]]` never reaches another vault's
  notes, and Obsidian warns against vaults nested inside vaults.

## Compatibility with Paniolo wikis

Our `wiki/` corpora are vault-shaped by construction: a flat directory of
`slug.md` files where `[[slug]]` means `slug.md`. Same-vault links and
`[[slug|alias]]` aliases are exactly Obsidian's syntax, so opening a wiki
root as a vault makes the link graph clickable with zero conversion.

Cross-wiki links are bare `[[slug]]` too: Paniolo tooling resolves a slug in
the page's own wiki first, then across the union of configured wikis. A vault
opened **above** the wiki roots does the same thing for the reader — Obsidian
finds `slug.md` wherever it sits, as long as slugs are globally unique (a few
generic names like `index` and `log` collide, and Obsidian picks one).

The retired **qualified link** used a colon — an invalid link character in
Obsidian that can never be a filename — so it was invisible to the reader and
is now a validation error. Write the bare slug and let the corpus resolve it.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
