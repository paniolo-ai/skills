---
name: paniolo-wiki
description: |
  Maintain a customer LLM wiki — ingest sources, author pages, lint, and reorganize pages. Use when adding or updating wiki pages, snapshotting raw sources, moving or deleting pages, or running wiki validation. Do not use for editing the wiki validator code itself (that is ordinary code work).
license: MIT
metadata:
  version: 0.5.14
tags:
- wiki
- llm-wiki
references: []
user-invocable: true
---

**Requires:** file-read, file-write, terminal (to run `pnpm run check:wiki` from harness).

# Maintaining a wiki

A pointer skill — the operating procedure, not a restatement of the rules. The
validator is the canonical schema: it enumerates the permitted values in its own
findings, so run it rather than guessing.

- `type:` — `entity`, `concept`, `summary`, `comparison`, `synthesis`,
  `decision`, `index`.
- `wiki/log.md` verbs — `ingest`, `query`, `lint`, `research`, `roadmap`,
  `decide`, `delete`, `rename`, `move`.
- Wikis, known repos, and thresholds — the `wiki` section of
  `paniolo.config.json`.

## Use When

- Ingesting a new source into `raw/<domain>/` and writing/updating `wiki/` pages.
- Answering a question from the wiki and optionally filing the answer back as a page.
- Linting the wiki for contradictions, stale claims, orphans, or missing cross-refs.
- Deleting, renaming, or moving a wiki page.

## Golden rule

Every factual claim must trace to a file you have snapshotted under `raw/`. If you
read it from another repo on disk, **snapshot it into `raw/` first** (and add it to
that directory's `SOURCES.md`) or do not claim it. The validator enforces that every
cited snapshot resolves — both `raw/<path>` entries and bare slugs — but it cannot
catch a fact you invented, so this discipline is on you.

Document anything that makes an AI agent more reliable across a long session:
harness structure, instruction-authoring techniques, enforcement/correction loops,
memory patterns, and distribution/remediation patterns. Prefer one concept per page;
split rather than sprawl.

## Ingest workflow

1. Copy the source verbatim into `raw/<domain>/<subdir>/` and add a row to that
   directory's `SOURCES.md` (File | Origin path | Repo commit | Date fetched).
2. Write or update the relevant `wiki/<domain>/` page(s). One concept per page;
   split rather than sprawl. Use wikilinks between wiki pages.
3. Add every required frontmatter field (`title`, `type`, `tags`, `updated`, and
   `sources` unless it is an `index` page). Use a real `YYYY-MM-DD` `updated` date.
4. List the new/changed page in the relevant category index, or in
   `wiki/<domain>/index.md` for small domains (avoids an orphan).
5. Append a `wiki/log.md` entry: `## [YYYY-MM-DD] <verb> | <domain> | <title>`.
6. **Run `pnpm run check:wiki` from the harness repo and fix everything it reports.**
   Do not finish red.

## Before contradicting an existing page

Search for the claim you are changing. If another page states the current behavior
in present tense, either update both or frame your change as a clearly-labeled
"future direction" — do not leave two pages disagreeing.

## Page operations

All references to wiki pages — in wiki pages, skills, agents, adapters, docs,
and sibling repo files — **must** use cross-wiki wikilink syntax. Use the wiki
name as it appears in `paniolo.config.json`:

```text
[[source-wiki:slug]]  or  [[source-wiki:slug|Display Text]]
```

Inside the wiki the page lives in, the prefix can be omitted. Do **not** use
markdown path references (`source-wiki/wiki/slug.md`) in any form.

### Delete

1. Grep all reference patterns for the slug across all configured wikis,
   `.agents/skills/`, `.agents/agents/`, `.github/`, `.devin/`, `.cursor/rules/`,
   root `*.md`, and sibling repos.
2. Update every reference: remove index list entries, replace inline references with
   the best alternative, remove markdown path references.
3. Remove the entry from every index that lists it — the alphabetical index
   (`all-pages.md`, where the wiki keeps one) and any parent/domain index.
4. Delete the file.
5. Append to `wiki/log.md`: `## [YYYY-MM-DD] delete | <domain> | <title>` with 1-2 bullets.
6. Run `pnpm run check:wiki` and fix remaining issues.

### Rename (same wiki)

1. Grep all reference patterns for the old slug.
2. Update every matching wikilink and markdown path, preserving any `|Display Text`.
   For a page in `source-wiki` this means local links and cross-wiki links like
   `source-wiki:old-slug` become `source-wiki:new-slug`, and markdown paths like
   `source-wiki/wiki/old-slug.md` become `source-wiki/wiki/new-slug.md`.
3. Rename the file; update frontmatter `title` if the title changes.
4. Update every index that lists it (`all-pages.md` where present, plus parent
   and domain indexes).
5. Log: `## [YYYY-MM-DD] rename | <domain> | <old> → <new>`.
6. Run `pnpm run check:wiki`.

### Move (between wikis)

1. Grep all reference patterns for the slug.
2. Update every matching wikilink and markdown path, preserving any `|Display Text`:
   - In the source wiki change local links to `target-wiki:<slug>`.
   - In the target wiki change `source-wiki:<slug>` to local links.
   - In other wikis change `source-wiki:<slug>` to `target-wiki:<slug>`.
   - Markdown paths update similarly.
3. Copy the file to the target wiki, delete from source.
4. Update indexes in both wikis.
5. If `sources:` cites `raw/<path>`, copy those into the target wiki's `raw/` tree
   and update `SOURCES.md` in both wikis.
6. Log in both wikis: `## [YYYY-MM-DD] move | <domain> | <title> → <target-wiki>`.
7. Run `pnpm run check:wiki`.

## Markdown lint best practices

Apply while writing so pages pass `pnpm run check:wiki` (and the harness `pnpm run lint`) on the first try.

- Blank line before AND after every heading, list, and fenced block.
- Unordered lists use `-`; ordered lists use `1.` for every item.
- Every fenced block has a language tag (`ts`, `bash`, `text`).
- Internal links resolve in your wiki; use backticks for cross-repo paths.
- Prose pages on the capped surface stay under the `wiki.lineCap` budget (350 by
  default). `type: index` pages are exempt — split them by domain when they grow.

## Do Not

- Do not cite a `raw/` path you have not actually snapshotted.
- Do not duplicate a fact across two pages — link to one canonical page.
- Do not leave a page out of its index (orphan) or skip the `wiki/log.md` entry.
- Do not finish while `pnpm run check:wiki` is red.
- Do not delete, rename, or move a file before updating its references.
- Do not move `raw/` sources without updating `SOURCES.md` in both wikis.
- Do not leave orphaned entries in any index that listed the page.
- Do not silently drop a reference — if no alternative exists, ask the human.
- Do not omit blank lines around headings, lists, or fences.
- Do not use bare URLs — always `[text](url)`.
- Do not exceed the configured line budgets (`skill.maxLines`, `wiki.lineCap`).

## References

- Validator: `pnpm run check:wiki` in harness (`paniolo wiki` via `@paniolo/cli`).
- Wiki config: `paniolo.config.json` — lists all wiki roots and known repos.