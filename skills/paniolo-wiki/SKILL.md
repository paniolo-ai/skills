---
name: paniolo-wiki
description: |
  Maintain a customer LLM wiki — ingest sources, author pages, lint, and reorganize pages. Use when adding or updating wiki pages, snapshotting raw sources, moving or deleting pages, or running wiki validation. Do not use for editing the wiki validator code itself (that is ordinary code work).
license: MIT
metadata:
  version: 0.5.15
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
- `wiki/log.md` verbs — see [Choosing a log verb](#choosing-a-log-verb).
- Wikis, known repos, and thresholds — the `wiki` section of
  `paniolo.config.json`.

## Choosing a log verb

Nine verbs, and the validator only rejects one it does not recognize — it cannot
tell you the verb you picked is the vaguest one that fits. Pick the narrowest:

| Verb | Use it when the entry records |
| --- | --- |
| `ingest` | A source was snapshotted into `raw/` and written up as page(s). |
| `research` | You investigated a question and filed what you learned. |
| `query` | You answered a question *from* the existing wiki. |
| `decide` | A choice was made, with its rationale and what it rules out. |
| `roadmap` | Planned or sequenced work that has not happened yet. |
| `lint` | Pages were corrected in place — contradictions, stale claims, orphans. |
| `delete` | A page was removed. |
| `rename` | A page's slug changed. |
| `move` | A page changed wikis. |

`lint` is the one that over-attracts, because "I fixed the wiki" describes a
delete, a rename, and a move too. If the entry's title names a page that stopped
existing, changed slug, or changed wiki, the verb is `delete`, `rename`, or
`move` — even though the work also involved cleanup. Reach for `lint` only when
the page set is unchanged and the content inside it was corrected.

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

**Do not grep-and-edit these by hand.** `paniolo wiki` performs all four.
`rename` and `move` write unless you pass `--dry-run`; `delete` is the reverse
and plans unless you pass `--apply`, because it is the one that destroys
information:

```bash
paniolo wiki refs <slug> --config paniolo.config.json
paniolo wiki rename <old> <new> --config paniolo.config.json --dry-run
paniolo wiki move <slug> --to <wiki> --config paniolo.config.json --dry-run
paniolo wiki delete <slug> --config paniolo.config.json            # plan only
paniolo wiki delete <slug> --config paniolo.config.json --apply
```

### Start with `refs`

It lists every reference and classifies each one:

- **index** — a bullet that exists only to point at the page. Removed for you.
- **prose** — the slug in a sentence. Blocks a delete; only you can pick what
  replaces it.
- **non-actionable** — generated or cached files (`generatedPaths` in config)
  and `wiki/log.md`, which is append-only history and is never rewritten.

All four operations also name any declared repo they could not search — a
`knownRepos` entry with no `repoPaths` mapping, or a mapping whose path no longer
resolves. Read that list before trusting an empty result: a repo that was never
opened looks exactly like a repo with no hits.

### Rename and move are fully automatic

Both rewrite every *link* across all configured repos: display aliases survive,
cross-wiki links keep their prefix through a rename, and a move retargets by
where the link lives (source wiki qualifies, destination localizes, everyone else
repoints). A move also carries the page's cited `raw/` snapshots into the target
wiki, and moves their `SOURCES.md` rows across verbatim — falling back to a note
when the row is ambiguous or the target table has a different shape.

Neither rewrites everything, and both say what they left:

- A **bare mention** — the slug in a sentence with no link syntax — survives a
  rename untouched, because `old-page` in prose may be a word rather than a
  reference. It is reported; deciding is yours.
- A **markdown path** survives a *move* untouched: the directory changed, and the
  right replacement depends on the referencing file's own depth. Reported too.
  (A rename does rewrite these — only the filename changes there.)

Then log it and validate:

- `## [YYYY-MM-DD] rename | <domain> | <old> → <new>`
- `## [YYYY-MM-DD] move | <domain> | <title> → <target-wiki>` (in both wikis)

### Delete plans, and holds the page back rather than deciding

Without `--apply` it writes nothing and prints the whole plan: the mechanical
edits, the prose references it will not touch, the hits it suppressed as
generated or historical, and any declared repo it could not search. Read the last
two — a wrong ignore glob or a stale `repoPaths` entry both look like "nothing
else points at it".

With `--apply` and prose outstanding, it removes the index entries but **keeps
the page** and exits non-zero, so the second run has a shorter list rather than
the same one. Resolve each prose site — pick the replacement page and rewrite the
sentence — then apply again; that run removes the file and validates what it
touched. `--json` emits the same plan for a caller to act on.

Log it (`## [YYYY-MM-DD] delete | <domain> | <title>`, 1-2 bullets), then run
`pnpm run check:wiki`.

Before deleting, check whether the page holds a still-valid fact absent from its
replacement. The command cannot know that; rescuing it is on you.

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
- Do not hand-edit references for a delete, rename, or move — use the commands.
- Do not force a delete past its refusal; resolve the prose references it names.
- Do not rewrite `wiki/log.md` entries for a renamed or deleted page — the log
  records what happened on a date.
- Do not log a page operation as `lint` — a title with `→` in it, or ending in
  "removed", is a `rename`, `move`, or `delete`.
- Do not move `raw/` sources without updating `SOURCES.md` in both wikis.
- Do not silently drop a reference — if no alternative exists, ask the human.
- Do not omit blank lines around headings, lists, or fences.
- Do not use bare URLs — always `[text](url)`.
- Do not exceed the configured line budgets (`skill.maxLines`, `wiki.lineCap`).

## References

- Validator: `pnpm run check:wiki` in harness (`paniolo wiki` via `@paniolo/cli`).
- Wiki config: `paniolo.config.json` — lists all wiki roots and known repos.