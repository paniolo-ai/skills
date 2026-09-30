---
name: paniolo-stale-remediate
description: |
  Drive the paniolo stale adjudication and remediation pipeline — run verifier/challenger agent phases, propose and merge remediation PRs, operate the durable worker, and calibrate detection with replay, shadow lanes, and canaries. Use when asked to run, adjudicate, remediate, propose, merge-sync, automate, or calibrate staleness work. To inspect the ledger without mutating, use paniolo-stale.
license: MIT
metadata:
  version: 0.1.4
tags:
- staleness
- remediation
- agents
- cli
user-invocable: true
references:
- references/stale-adjudication.md
- references/stale-automation.md
- references/stale-calibration.md
- references/stale-commands.md
- references/stale-configuration.md
- references/stale-ledger.md
- references/stale-lifecycle.md
- references/stale-triggers.md
---

# paniolo-stale-remediate — run the adjudication pipeline

`paniolo stale` moves `pending-verification` allegations through
independent agent roles — verifier, verdict challenger, remediator, patch
challenger — and publishes accepted fixes as PRs. This skill drives those
mutating phases. Read
[stale-adjudication](references/stale-adjudication.md) for the role,
adapter, disposition, and merge-gate contract.

## Before Running

- The configured agent CLIs (`codex`, `claude`, `cursor`, `devin`) must be installed
  and signed in — preflight runs `<cli> --version` per role and stops
  cleanly if one fails. Roles, models, and budgets come from the
  `staleness` config; see
  [stale-commands](references/stale-commands.md) for the override flags.
- `propose`, `merge-sync`, and `worker` need `git` and an authenticated
  `gh`. They create real branches and PRs — say so to the user before
  running them.
- Prefer `--dry-run` on `propose`, `merge-sync`, and `worker` to read the
  plan before the first real run.
- Check which config the invocation selects and whether
  `staleness.enabled` permits the phase. A successful
  `"status": "disabled"` response means no work ran; do not bypass it or
  report completion. See [stale-triggers](references/stale-triggers.md).

## Workflow

1. Adjudicate: `paniolo stale run --repo <key>:<path> [--adapter ...]
   [--model ...]`. The verifier and challenger settle each pending
   allegation; confirmed-stale work then gets a remediator patch and a
   patch-challenger check. Watch for `stopped: "preflight …"` in the JSON.
2. Inspect results with `stale list --actionable` and `stale show <id>`;
   allegations that could not be grounded stay `insufficient-evidence` —
   durable work, not failure.
3. Publish fixes: `propose --repo <key>:<path> --dry-run`, then without
   `--dry-run` to push `staleness/rem-*` branches and open PRs.
4. Reconcile: `merge-sync` promotes merged heads through the merge gate and
   clears closed proposals for re-queue.
5. For continuous operation, use `worker --ledger-repo <key>:<path>
   --repo ... [--wiki ...] [--bootstrap]` — it merge-syncs, scans from
   scan checkpoints, adjudicates, proposes, and publishes the ledger as a PR on
   `staleness/ledger`. One run per ledger via `run.lock`.

## Lifecycle Lane

`scan --wiki <key>:<repo-root>` also nominates `LC-` lifecycle candidates
for `plan-`/`design-` pages whose work has shipped. The same `run` →
`propose` → `merge-sync` loop moves them `pending-review → applied`. The
page legibility contract — verdict plus evidence pointer on one status
line, gate verdicts in a `Result` column — decides whether a page can
nominate at all; see
[stale-lifecycle](references/stale-lifecycle.md) before treating a quiet
scan as "nothing to do".

## Calibration

- `seed <target> <claim> --expected stale|fresh` plants a known-answer
  canary through normal adjudication.
- `replay` runs built-in known-answer cases through real adapters — always
  against a disposable `--root`; it writes ledger records.
- `shadow-qmd <out.json>` measures the qmd retrieval lane over the holdout
  corpus; `replay --full --shadow-observations <file>` scores lane
  admission; `score-shadow <file>` re-validates a file standalone. A missing
  or invalid qmd index-integrity manifest fails scoring rather than
  creating a negative retrieval label.
- `staleness.retrieval.shadow.enabled: true` records qmd document and
  section funnels during live `scan` and `worker` cycles. It remains
  non-authoritative: no fuzzy allegation, agent call, disposition, or merge
  authority.
- `report` recomputes per-surface outcomes, canary hits/misses, and the
  false-resolution budget. See
  [stale-calibration](references/stale-calibration.md).

## Do Not

- Do not hand-edit ledger JSON or force transitions — the state machine and
  merge gate exist so dispositions stay evidence-based.
- Do not bypass the merge gate or delete `automerge.disabled` to make a
  merge happen; investigate what the gate rejected.
- Do not point `replay` at a live ledger.
- Do not promote a fuzzy or weighted-routing lane on an invalid index,
  unlabeled tail, or a report that only counts its own agent filings.
- Do not export secrets hoping to reach agent CLIs — invocations run with a
  scrubbed environment; fix the CLI's own sign-in instead.
- Do not delete `.stale-rem-*` or `.stale-ledger-*` worktrees while their
  branches or runs may be live — check `git worktree list` first. See
  [stale-ledger](references/stale-ledger.md).

## References

- [stale-adjudication](references/stale-adjudication.md) — roles, adapters,
  dispositions, merge gate, worker, CI advisory
- [stale-automation](references/stale-automation.md) — the loop, surfaces,
  watches, and lifecycle
- [stale-triggers](references/stale-triggers.md) — automation entry points,
  enablement, config authority, and deployment recipes
- [stale-calibration](references/stale-calibration.md) — corpus, gates,
  replay, shadow lanes, canaries, report
- [stale-commands](references/stale-commands.md) — `run`, `propose`,
  `merge-sync`, `worker` flag reference
- [stale-configuration](references/stale-configuration.md) — the
  `staleness` config schema, validation, and CLI overrides
- [stale-ledger](references/stale-ledger.md) — sealed observations,
  bundles, proposals, locks
- [stale-lifecycle](references/stale-lifecycle.md) — `LC-` candidates,
  page legibility grammar, status-patch proposals

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
