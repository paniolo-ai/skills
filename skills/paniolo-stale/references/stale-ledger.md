---
source-slug: stale-ledger
source-hash: 44ca8a4a6b732339ae86cb64c94d0470e5bdb907b78bb7d2717e6fad8833a2d7
bundled: 2026-09-27
title: Stale Ledger
type: concept
tags:
- staleness
- harness-eng
- ledger
updated: 2026-09-27
---

# Stale Ledger

The ledger is the durable state behind `paniolo stale`: a directory of
JSON records inside the repository that owns the prose. Wiki-page
allegations live in the wiki repository; code-comment allegations live in
the code repository; workspace commands aggregate the local stores.

## Contents

- [Ledger Location](#ledger-location)
- [Directory Layout](#directory-layout)
- [Record Ids](#record-ids)
- [State Machine](#state-machine)
- [Cursors And Pending Scans](#cursors-and-pending-scans)
- [Worktrees And Branches](#worktrees-and-branches)
- [Store Invariants](#store-invariants)
- [See Also](#see-also)

---

<a id="ledger-location"></a>

## Ledger Location

Resolution order:

1. `--config`'s `staleness.ledgerPath` (or the default
   `.paniolo/staleness`), resolved under `--root`.
2. Legacy fallback, only when no `--config` was passed and the file has no
   `staleness` section: an existing `<root>/staleness/` directory is used
   while the configured path does not exist.

The resolved path must stay inside the checkout — absolute paths, `..`
segments, drive prefixes, and symlinked roots are rejected. Moving a ledger
is an explicit migration: locks, cursors, observations, proposal records,
and git history form one durable whole.

---

<a id="directory-layout"></a>

## Directory Layout

```text
<ledgerPath>/
  allegations/         S-<hash>.json   mutable; state machine + revision-checked
  evidence/            E-<hash>.json   immutable, content-addressed
  labels/              L-<hash>.json   immutable; corrections supersede
  retrieval-runs/      R-<hash>.json   bounded candidate pools, incl. deferred
  observations/        O-<hash>.json   immutable sealed agent outputs by role
  remediation-bundles/ B-<hash>.json   pins the four observation ids + patch hash
  proposals/           <remote>.json   pending candidate-PR proposal per remote
  cursor.json          per-repo last materialized scan heads
  pending-scans.json   heads carried by an unmerged ledger PR
  state.lock           exclusive lock for every mutable read-modify-write
  run.lock             worker-only single-run mutex
  automerge.disabled   kill-switch sentinel: blocks every merge decision
```

All writes are atomic (temp file + rename in the same directory) and every
record path is validated to stay under the ledger root.

---

<a id="record-ids"></a>

## Record Ids

Ids are `sha256` truncated to 16 hex characters, computed over fields joined
with `0x1f`. The prefix names the record kind:

| Prefix | Record | Id inputs |
| --- | --- | --- |
| `S-` | Allegation | location id + detector + claim |
| `E-` | Evidence | source repo + commit + changed entity + detector version + allegation id |
| `X-` | Excerpt | cited evidence slice |
| `C-` | Change | detected change record |
| `L-` | Label | human/agent gold label |
| `R-` | Retrieval run | one scan's candidate pool |
| `O-` | Observation | one sealed agent response |
| `B-` | Remediation bundle | verifier, challenger, remediator, patch-challenger observation ids + patch hash |

Because ids are content-addressed, re-scanning an overlapping range rewrites
identical bytes and reopens retained work rather than duplicating it. A
different claim at the same location is a different allegation. Commands
accept a `S-` id or a unique prefix.

---

<a id="state-machine"></a>

## State Machine

Allegations mutate only through a checked transition with an optimistic
`revision`; a stale writer fails instead of overwriting newer evidence.

| From | May transition to |
| --- | --- |
| `pending-verification` | `confirmed-stale`, `dismissed`, `insufficient-evidence`, `obsolete` |
| `confirmed-stale` | `remediation-proposed`, `dismissed`, `insufficient-evidence`, `obsolete` |
| `remediation-proposed` | `resolved-updated`, `dismissed`, `insufficient-evidence`, `obsolete` |
| `insufficient-evidence`, `dismissed`, `resolved-updated` | `pending-verification`, `obsolete` |
| any open state | `pending-verification` on a content edit at the location |

`insufficient-evidence` is durable: the work is retained and re-enters
verification on new evidence or `--retry-retained`, never abandoned.

---

<a id="cursors-and-pending-scans"></a>

## Cursors And Pending Scans

`cursor.json` maps each repository key to the last materialized scan head —
the worker scans from cursor to HEAD each cycle. Scan heads recorded while
a ledger PR is still open go to `pending-scans.json`; they promote to
`cursor.json` only when the ledger PR carrying them merges. Remediation
merges do not move cursors — only merged ledger PRs do. `worker
--bootstrap` seeds each `--repo` cursor at HEAD without scanning history.

---

<a id="worktrees-and-branches"></a>

## Worktrees And Branches

Remediation and ledger publishing happen in sibling worktrees so the main
checkout and its hooks are untouched:

- `.stale-rem-<short>` — created beside the checkout for `propose`/`worker`
  remediation. `short` is characters 2–10 of the first allegation id. The
  worktree gets branch `staleness/rem-<short>`, the sealed patches (applied
  through the engine's own parser, not `git apply`), a `commit-tree`
  plumbing commit (hooks bypassed deliberately), a push, and `gh pr
  create`. The worktree is removed afterward; the orphan branch is deleted
  on failure. Leftover empty `.stale-rem-*` directories are cleanup residue
  from earlier runs — check `git worktree list` before deleting one whose
  branch may still exist.
- `.stale-ledger-<key>` — the worker's persistent worktree for the
  `staleness/ledger` branch. A healthy one is reused so unpublished
  transitions survive between runs; `--dry-run` uses a detached HEAD.
- `staleness/ledger` branch — where the ledger itself is published as a
  candidate PR. Nothing lands without merge.

Remediation pushes run with a private `CARGO_TARGET_DIR` under the worktree
so pre-push gate builds do not poison shared caches.

---

<a id="store-invariants"></a>

## Store Invariants

- Evidence is write-once: identical bytes under an existing `E-` id are a
  no-op; different bytes are an error.
- Labels are immutable; a correction is a new label naming its `supersedes`
  target.
- Observations are sealed immutable records bound to a cycle; a remediation
  bundle pins the exact four observation ids and the patch hash it applies.
- An allegation's revision increments on every transition, so concurrent
  writers fail closed.
- Migration never discards unresolved state: an ambiguous anchor mapping
  retains the allegation and records the ambiguity.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
