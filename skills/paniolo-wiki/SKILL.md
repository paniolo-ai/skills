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
user-invocable: true
references: []
---

**Requires:** file-read, file-write, terminal (to run `paniolo wiki` from harness).

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

## Naming the page

The leading token of a slug names the page's **kind** — a plan, a decision, a
paper digest, a blog digest, a third-party repo digest. Pick it before you
write: the slug is what wikilinks, file lists, and search results carry, and
changing it later costs a rename across every repo.

**The validator does not check this.** `check:wiki` passes a mis-prefixed page,
so a wrong slug survives until a human reads the directory listing. The prefix
table is authored per wiki, on that wiki's own naming-conventions page — read
it before naming a new page. If the wiki has no such page, match the sibling
pages already in that domain and say which ones you matched.

Three ways it goes wrong:

- **Prefixing by topic instead of kind.** A page *about* planning technique is
  a concept page, not a plan; a page digesting a paper about blogging is a
  paper digest. Ask what the page **is**, not what it discusses.
- **Stacking two kind prefixes.** A page has one kind. When a source is both a
  repo and a paper, the prefix follows the thing the page actually digests.
- **Prefixing a catalog.** An index listing many papers is not a paper digest
  and keeps its own descriptive slug. `type: index` is usually the tell.

A rename must land on a correctly prefixed slug. `paniolo wiki rename` rewrites
links to whatever target you hand it and will not tell you the target is wrong.

## Creating a page

`paniolo wiki new` stamps the page, its frontmatter, the `wiki/log.md` entry, and
one index link in a single deterministic step. Use it instead of hand-writing a
file — the log entry and index link are the two things hand-creation forgets.

```bash
paniolo wiki new <slug> --config paniolo.config.json \
  --type synthesis --verb roadmap --tags a,b --source raw/<domain>/<file>.md \
  --index <index-slug>
```

**Prefer `--kind`** when the wiki declares kinds under its own entry in the
config's `wiki.wikis[].pagePrefixes`. Kinds are declared per wiki, not
workspace-wide — one corpus may file plans and decisions while another files
neither. One flag supplies the slug prefix, the `type:`, the log verb, the
starting `status:`, and the indexes to link into:

```bash
paniolo wiki new <slug> --kind plan --config paniolo.config.json
# → plan-<slug>.md, type/verb/status from the kind, indexes wired
```

A slug that already carries a known prefix resolves its kind on its own, so
passing `decision-<slug>` is equivalent to `<slug> --kind decision`. Any
explicit flag still beats the kind.

Without a kind the defaults are `--type concept` and `--verb ingest`, **which
are wrong for every prefixed kind** — pass the type and log verb yourself, or you
get a `concept`/`ingest` page with the right name and the wrong metadata. It
writes frontmatter and an H1 and stops; the body is yours.

## Recording status

Kinds that track a lifecycle declare their vocabulary in that wiki's config
entry, and `status:` must come from it. Use the wiki's own values; do not invent
a synonym. The four
words for "finished" and three for "underway" that a corpus accumulates are not
a style problem — they make the field unsearchable, and the validator only
catches them once the vocabulary is declared.

The value that matters most is the one for a page no longer worth acting on. A
plan that was shipped or abandoned with no way to say so reads as live guidance,
and an agent retrieving it will act on work that is already over.

Set it at creation (`--kind` supplies the starting value, `--status` overrides)
and update it when the state changes — a `status:` that stopped being true is
worse than none, because it is believed.

## Ingest workflow

1. Copy the source verbatim into `raw/<domain>/<subdir>/` and add a row to that
   directory's `SOURCES.md` (File | Origin path | Repo commit | Date fetched).
2. Write or update the relevant `wiki/<domain>/` page(s). Name new pages by kind
   (see [Naming the page](#naming-the-page)) and stamp them with `paniolo wiki
   new` (see [Creating a page](#creating-a-page)). One concept per page; split
   rather than sprawl. Use wikilinks between wiki pages.
3. Add every required frontmatter field (`title`, `type`, `tags`, `updated`, and
   `sources` unless it is an `index` page). Use a real `YYYY-MM-DD` `updated` date.
4. List the new/changed page in the relevant category index, or in
   `wiki/<domain>/index.md` for small domains (avoids an orphan).
5. Append a `wiki/log.md` entry: `## [YYYY-MM-DD] <verb> | <domain> | <title>`.
6. **Run `paniolo wiki` from the harness repo and fix everything it reports.**
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
`paniolo wiki`.

Before deleting, check whether the page holds a still-valid fact absent from its
replacement. The command cannot know that; rescuing it is on you.

## Markdown lint best practices

Apply while writing so pages pass `paniolo wiki` (and the harness the project's lint script) on the first try.

- Blank line before AND after every heading, list, and fenced block.
- Unordered lists use `-`; ordered lists use `1.` for every item.
- Every fenced block has a language tag (`ts`, `bash`, `text`).
- Internal links resolve in your wiki; use backticks for cross-repo paths.
- Prose pages on the capped surface stay under the `wiki.lineCap` budget (350 by
  default). `type: index` pages are exempt — split them by domain when they grow.

## Do Not

- Do not cite a `raw/` path you have not actually snapshotted.
- Do not name a page without checking the wiki's prefix convention — the
  validator is green either way, so a wrong slug is invisible to `check:wiki`.
- Do not rename onto an unprefixed slug; the rename command accepts any target.
- Do not duplicate a fact across two pages — link to one canonical page.
- Do not leave a page out of its index (orphan) or skip the `wiki/log.md` entry.
- Do not finish while `paniolo wiki` is red.
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

- Validator: `paniolo wiki` in harness (`paniolo wiki` via `@paniolo/cli`).
- Wiki config: `paniolo.config.json` — lists all wiki roots and known repos.
