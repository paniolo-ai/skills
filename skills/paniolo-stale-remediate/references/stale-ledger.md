---
source-slug: stale-ledger
source-hash: 5dfdb1a518be6b5174f0981fec15a9a39a8e04178a0a60ccc175cf19baab2dfb
bundled: 2026-10-11
title: Stale Ledger
type: concept
tags:
- staleness
- harness-eng
- ledger
updated: 2026-10-11
---

# Stale Ledger

The ledger is the durable state behind `paniolo stale`. Its live records
are in the checkout owner's platform-native `workspace.sqlite`, shared
with qmd under separate schema owners. Git carries a stale-only recovery
snapshot at `.paniolo/ledger.sqlite` and the writer lease at
`.paniolo/writer.json`. A finding's repository remains part of its record
identity even when another repository's diff detected it.

## Contents

- [Ledger Location](#ledger-location)
- [Storage Layout](#storage-layout)
- [Record Ids](#record-ids)
- [Agent-Reported Observations](#agent-reported-observations)
- [State Machine](#state-machine)
- [Scan Checkpoints And Pending Scans](#scan-checkpoints-and-pending-scans)
- [Worktrees And Branches](#worktrees-and-branches)
- [Store Invariants](#store-invariants)
- [See Also](#see-also)

---

<a id="ledger-location"></a>

## Ledger Location

The live database is selected by the owner checkout, not by `ledgerPath`.
Windows and WSL keep separate native files and coordinate through the
published snapshot and writer lease. A missing or incompatible live
database fails closed; it never becomes an empty JSON ledger.

`staleness.ledgerPath` names the repository-relative publication directory
(default `.paniolo`). Absolute paths, `..` segments, drive prefixes, and
symlinked roots are rejected. `stale migrate` is the explicit importer for
an existing legacy JSON tree.

---

<a id="storage-layout"></a>

## Storage Layout

```text
<native state>/<owner>/workspace.sqlite
  stale_* tables         durable ledger records and lineage
  ops_* tables           rebuildable jobs and metrics; not in the snapshot
  qmd-owned tables       local retrieval index; not in the snapshot
<ledger checkout>/.paniolo/
  ledger.sqlite          Git-tracked stale-only recovery snapshot
  writer.json            Git-tracked cross-platform writer lease
  managed.lock           local scheduled-worker mutex
```

The `stale_*` tables hold allegations, evidence, labels, retrieval runs,
agent reports, observations, proposals, references, lifecycle records,
and singleton documents for scan checkpoints and pending scans. Records
carry repository keys; a record with no attributable repository fails
closed. Writes use SQLite transactions and checked revisions. Snapshot
export copies only durable stale-owned state and stamps its generation.
The native run lock and journal sit beside `workspace.sqlite`.

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
| `L-` | Label | Evaluation fact with provenance-constrained gold, silver, or outcome strength |
| `R-` | Retrieval run | one scan's candidate pool |
| `A-` | Agent report | target + claim + encountered observation + optional source revision |
| `O-` | Observation | one sealed agent response |
| `B-` | Remediation bundle | verifier, challenger, remediator, patch-challenger observation ids + patch hash |
| `RI-` | Reference index | repo + prose file path; rewritten in place per scan |
| `LC-` | Lifecycle candidate | wiki + page slug |
| `LE-` | Lifecycle evidence | candidate id + evidence source |
| `LB-` | Lifecycle bundle | candidate id + patch hash |

Because ids are content-addressed, re-scanning an overlapping range rewrites
identical bytes and reopens retained work rather than duplicating it. A
different claim at the same location is a different allegation. Commands
accept a `S-` id or a unique prefix.

---

<a id="agent-reported-observations"></a>

## Agent-Reported Observations

`paniolo stale flag` stores a bounded `observed-in-use/1` report in the
ledger's agent-report records. It records the exact `repo:path#anchor`, relied-on claim,
observation file contents, and optional source revision. Refiling the same
packet is idempotent; a different observation remains distinct. Filing
does **not** create an allegation, verdict, or remediation request.

The dogfood report measures this lane separately. It matches a filed
report to an independently verified outcome only when both the location
and trimmed claim match. Confirmed-stale and dismissed-fresh matches form
the precision denominator; reports with no matching outcome remain
unknown, not false. Repeated reports never become gold labels.

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

<a id="scan-checkpoints-and-pending-scans"></a>

## Scan Checkpoints And Pending Scans

A **scan checkpoint** is the last source commit whose detection results were
successfully materialized into a merged ledger. A `stale_documents` row maps
each repository key to that commit, and the worker scans the range
`checkpoint..HEAD` each cycle. Pending scan heads are stored in another
singleton document while a ledger PR is open; they become checkpoints only
when the ledger PR carrying them merges. Remediation merges do not move scan
checkpoints. `worker --bootstrap` seeds each `--repo` checkpoint at HEAD
without scanning history.

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
- `.stale-ledger-<key>` — the worker's candidate worktree for the
  `staleness/ledger` branch. A healthy one is reused for publication;
  unpublished transitions remain in the owner's native database.
  `--dry-run` uses a detached HEAD.
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
