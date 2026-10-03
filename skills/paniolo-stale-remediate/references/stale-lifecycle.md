---
source-slug: stale-lifecycle
source-hash: 14631273dc0a817a36b01005458cd2f1e35fdae45510159661eb38fc9c233816
bundled: 2026-10-02
title: Stale Lifecycle
type: concept
tags:
- staleness
- harness-eng
- agents
- cli
updated: 2026-09-30
---

# Stale Lifecycle

The lifecycle lane tracks a wiki page's declared status through `LC-`
candidates — separate from prose-staleness `S-` allegations. An allegation
says "this claim may be stale"; a lifecycle candidate says "this page's
status is wrong because its work finished or it was superseded".

## States

`pending-review → verified-transition → transition-proposed → applied`.
`declined` and `insufficient-evidence` are durable open outcomes — never
forced. Editing the page while a candidate is open reopens it at
`pending-review` via revision tracking.

## Detection Grammar

- `plan-*` pages nominate `completed`; `design-*` pages nominate
  `implemented`, `superseded`, or `obsolete`. A page is eligible only when
  the wiki config declares its prefix and the proposed status exists in
  that prefix's vocabulary.
- Plan cards are `###` headings under a `##` section named one of:
  Implementation Cards, Implementation Phases, Task Queue, Cards, Work
  Items, Ordered Work. Other sections stay prose.
- A card resolves only when a `Status:`, `Status and evidence:`, or
  `Status/evidence:` line carries the verdict **and an evidence pointer on
  the same line**. `Implemented —` alone is a bare done-word; evidence on
  continuation lines is invisible to the detector.
- Tables under a `##` Status / Verification heading: `| Card | Status |`-keyed
  rows re-declare card verdicts (the worst declared outcome wins);
  `| Gate | Cards |` rows are extra obligations. A gate stays open when it
  names no declared card, references an unknown card id, covers an
  unresolved card, or declares an unresolved verdict in a `Status` or
  `Result` column. Rows with no verdict column defer to their cards.
- Deferred or descoped cards are recorded as obligations — counted, never
  disguised as done.

## Command Sequence

```text
scan --wiki key:repo-root --base <sha> --head <sha>   # nominate LC-
run --repo key:path                                  # verifier + challenger
propose --repo key:path [--dry-run]                  # status patch → PR
merge-sync --repo key:path [--dry-run]               # merge gate → applied
report                                             # dogfood summary
```

`run` adjudicates `pending-review` candidates with the configured verifier
and challenger — one `LE-`-bound packet per candidate, a sealed
verifier→challenger pair. `propose` renders the shared status operation in
an isolated worktree, challenges the exact diff, runs the worktree wiki
gate, then branches, pushes, and opens the PR. `merge-sync` watches the
proposal head: a merged PR applies every member through the merge gate, a
closed one clears the proposal.

## Operational Contract

- **Commit page edits before `propose`.** The proposal worktree branches
  from `HEAD` and re-verifies page bytes; an uncommitted edit reopens the
  candidate on revision drift instead of shipping a patch over bytes the
  verifier never saw.
- **`insufficient-evidence` is not failure.** When a verifier cannot ground
  an obligation in the packet's excerpts, fix the page's legibility — move
  evidence pointers onto the verdict line, give gate rows a `Result`
  column — rescan, and the candidate re-adjudicates.
- The shipped patch is confined to the `status:`/`updated:` frontmatter
  pair plus an append-only `log.md` entry; the patch challenger refuses
  any wider diff.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
