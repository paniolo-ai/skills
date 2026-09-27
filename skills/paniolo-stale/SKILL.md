---
name: paniolo-stale
description: |
  Operate the paniolo stale staleness ledger — scan a commit range for prose the code may have invalidated, inspect and triage allegations, manage the queue, and read the dogfood report. Use when asked to scan for stale docs, list or show staleness findings, check the staleness queue, or triage allegations. Read-mostly — for agent verification, remediation PRs, and the durable worker use paniolo-stale-remediate.
license: MIT
metadata:
  version: 0.1.1
tags:
- staleness
- ledger
- cli
- docs
user-invocable: true
references:
- references/stale-adjudication.md
- references/stale-automation.md
- references/stale-calibration.md
- references/stale-commands.md
- references/stale-configuration.md
- references/stale-ledger.md
- references/stale-triggers.md
- references/stale-unique.md
---

# paniolo-stale — operate the staleness ledger

`paniolo stale` maintains a durable queue of allegations that a code change
invalidated prose — wiki pages, ordinary docs, and code comments. Detection
nominates work but never proves truth: every disposition is evidence-based.
Read [stale-automation](references/stale-automation.md) for the model,
[stale-commands](references/stale-commands.md) for the full verb contract,
and [stale-unique](references/stale-unique.md) when asked what the feature
is for or how it pairs with qmd.

## Before Running

- Confirm the binary carries the `stale` feature (`paniolo stale --help`);
  it ships in `release-core` but not every installed build has it.
- Identify `--root` (the checkout that owns the ledger) and `--config`
  (defaults to `paniolo.config.json` under the root). The `staleness`
  section is documented in
  [stale-configuration](references/stale-configuration.md).
- Check the effective `staleness.enabled` policy and which config the
  invocation selects. A successful `"status": "disabled"` response means
  the command did not run; never report it as an empty scan. Read
  [stale-triggers](references/stale-triggers.md) for config and trigger
  precedence.
- Know whether you may mutate: this skill covers inspection and queue
  bookkeeping. Agent runs, proposals, merges, and the worker belong to
  `paniolo-stale-remediate`.

## Workflow

1. Scan a range without writing with `paniolo stale scan`. Put the global
   `--root <repo>` option before `scan`, then supply `--code <key>:<path>`,
   `--base <sha>`, `--head <sha>`, and `--dry-run` to the scan. Add
   `--wiki <key>:<path>` for declared watches on wiki checkouts.
2. Inspect the queue with `list --actionable`, `next`, and `show <id>` —
   `show` prints the allegation plus its sealed observations by role.
3. Read the deterministic dogfood report with `report` — per-surface
   outcomes, canary hits/misses, and `automerge_recommended`.
4. Triage with the queue verbs: `retry <id>` requeues insufficient or
   dismissed work, `resolve <id> --note <ref>` closes a proposal whose fix
   landed elsewhere, `rebase` re-resolves locations, and `prune` removes
   terminal records. Run `rebase` and `prune` without `--apply` first —
   both print a plan.

## Guardrails

- `list`, `next`, `show`, `report`, and `score-shadow` are read-only.
  `scan --dry-run`, `propose --dry-run`, `merge-sync --dry-run`, and
  `worker --dry-run` are plan-only. `prune` and `rebase` write only with
  `--apply`; `shadow-qmd` writes its requested output file, not the ledger.
- Allegations move through a checked state machine — use the verbs rather
  than editing `S-*.json` files by hand. See
  [stale-ledger](references/stale-ledger.md) for the layout.
- `.stale-rem-*` and `.stale-ledger-*` directories beside a checkout are
  remediation worktrees, not scratch space — check `git worktree list`
  before deleting one.
- Output is JSON on stdout; a nonzero exit means an error, not a finding.

## Do Not

- Do not run `replay` against a live ledger — it writes records; use a
  disposable `--root`.
- Do not run `propose`, `merge-sync`, or `worker` from this skill — they
  create PRs and push branches (see `paniolo-stale-remediate`).
- Do not treat a `dismissed` allegation as deleted — it is retained history;
  `prune` is what removes terminal records.
- Do not assume `insufficient-evidence` is closed — it is durable open work
  that re-enters verification on new evidence.

## References

- [stale-automation](references/stale-automation.md) — the loop, surfaces,
  declared watches, and lifecycle
- [stale-triggers](references/stale-triggers.md) — manual, CI, worker, retry,
  and calibration triggers plus enablement precedence
- [stale-commands](references/stale-commands.md) — every subcommand, flag,
  and output convention
- [stale-configuration](references/stale-configuration.md) — the
  `staleness` config schema, validation, and CLI overrides
- [stale-ledger](references/stale-ledger.md) — directory layout, record
  ids, state machine, cursors, worktrees
- [stale-adjudication](references/stale-adjudication.md) — agent roles,
  dispositions, merge gate, worker, CI advisory
- [stale-calibration](references/stale-calibration.md) — corpus, gates,
  replay, shadow lanes, canaries, report
- [stale-unique](references/stale-unique.md) — why the feature exists and
  how it combines with qmd

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
