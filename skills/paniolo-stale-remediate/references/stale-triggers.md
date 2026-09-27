---
source-slug: stale-triggers
source-hash: 19f7997bd3908d47b9a3fc096251ff55827ea561124a4ba4d5b33ff1d5469def
bundled: 2026-09-27
title: Stale Triggers
type: concept
tags:
- staleness
- harness-eng
- automation
- configuration
updated: 2026-09-27
---

# Stale Triggers

A trigger is the event and policy boundary that starts one part of the
staleness loop. `paniolo stale` supplies commands and a durable worker, but it
does not schedule itself: an operator, CI workflow, or external scheduler must
invoke the command with an explicit configuration.

## Contents

- [Trigger Matrix](#trigger-matrix)
- [Configuration Authority](#configuration-authority)
- [Automation Switch](#automation-switch)
- [Pull-Request Advisory](#pull-request-advisory)
- [Durable Worker](#durable-worker)
- [Retries And Reopenings](#retries-and-reopenings)
- [Safety Controls](#safety-controls)
- [Deployment Recipes](#deployment-recipes)
- [See Also](#see-also)

---

<a id="trigger-matrix"></a>

## Trigger Matrix

| Trigger | Starts when | Command | Agents | Durable or external writes |
| --- | --- | --- | --- | --- |
| Manual advisory | Operator supplies `base` and `head` | `scan --dry-run` | No | None; JSON report only |
| Manual detection | Operator supplies `base` and `head` | `scan` | No | Ledger allegations and retrieval runs |
| Pull-request advisory | CI receives a PR event | `scan --dry-run` | No | CI summary and artifact only |
| Manual adjudication | Operator elects to process the queue | `run` | Yes | Ledger observations, transitions, and bundles |
| Proposal publication | Operator elects to publish accepted fixes | `propose` | No | Worktree, branch, push, PR, proposal record |
| Proposal reconciliation | Operator or worker checks GitHub | `merge-sync` | No | Ledger transitions after GitHub reads |
| Durable cycle | Scheduler or operator invokes it | `worker` | Yes | Ledger, branches, PRs, and ledger PR |
| Retained retry | Operator requests another attempt | `retry` or `worker --retry-retained` | Later | Allegation transition |
| Calibration | Operator starts an experiment | `seed`, `replay`, `shadow-qmd` | Varies | Ledger records or shadow JSON file |

`next`, `list`, `show`, `report`, and `score-shadow` inspect existing state;
they do not trigger detection or remediation.

---

<a id="configuration-authority"></a>

## Configuration Authority

Every invocation has one effective configuration:

1. `--config <path>` selects that file. A relative path resolves under
   `--root`.
2. Without `--config`, the command loads `<root>/paniolo.config.json` when it
   exists; otherwise it uses built-in defaults.
3. When the selected file is canonically named `paniolo.config.json`, a sibling
   `paniolo.config.local.json` recursively overlays it. Objects merge, arrays
   replace, and the local file stays untracked.
4. A differently named explicit config is independent. It does not inherit the
   canonical config or its local overlay.
5. Command flags override configured roles and budgets for that invocation.

A workflow using `--config .github/staleness-advisory.json` therefore has its
own enablement and scope. Changing the root config does not change that
workflow unless the workflow is updated to use it.

---

<a id="automation-switch"></a>

## Automation Switch

`staleness.enabled` gates the entry points that nominate, judge, or publish
work: `scan`, `run`, `propose`, and `worker`.

When disabled, those commands exit successfully with JSON containing
`"status": "disabled"` and make no ledger or external changes. Callers must
treat that response as **not run**, not as an empty successful scan.

Inspection and explicit maintenance remain available while automation is off:
`list`, `next`, `show`, `report`, `score-shadow`, `retry`, `resolve`, `prune`,
`rebase`, `merge-sync`, `seed`, and `replay`. `shadow-qmd` runs before the
ledger configuration is opened and is also unaffected by `enabled`.

---

<a id="pull-request-advisory"></a>

## Pull-Request Advisory

The Ranch Hand `staleness-advisory` workflow runs on each pull request. It
scans `base..head` with `--dry-run`, publishes a bounded summary, and uploads
`staleness-report`. It has read-only repository permissions, no agent
credentials, and cannot block a merge.

The workflow passes `.github/staleness-advisory.json`. That file currently
enables its advisory independently of the harness root configuration. Teams
that want no shared staleness signal must disable or remove that workflow
policy as well as disabling the canonical config.

---

<a id="durable-worker"></a>

## Durable Worker

`worker` is durable orchestration, not a daemon or scheduler. One invocation
runs merge-sync, scans each repository from its saved scan checkpoint to HEAD,
adjudicates, proposes fixes, and publishes the ledger branch. Then it exits.

A local task runner, self-hosted scheduler, or operator must invoke it again.
Only a merged ledger PR advances scan checkpoints, so missed or failed
invocations retain their range. `run.lock` prevents two worker invocations
from owning the same ledger concurrently.

Use `worker --bootstrap` once to record each repository's current HEAD without
scanning its history. Use `--retry-retained` when a cycle should requeue
`insufficient-evidence` work before adjudication.

---

<a id="retries-and-reopenings"></a>

## Retries And Reopenings

The queue has its own data-driven triggers:

- New evidence or a content edit reopens work at that location as
  `pending-verification`.
- `retry <id>` explicitly requeues work allowed by the state machine.
- `worker --retry-retained` requeues all `insufficient-evidence` work before
  the cycle.
- A closed remediation PR clears its proposal during `merge-sync`, allowing
  the allegation to re-enter the queue.
- A stale target hash or patch context requeues the allegation instead of
  force-applying it.

These triggers preserve unknown work; none treats delay or disagreement as a
fresh verdict.

---

<a id="safety-controls"></a>

## Safety Controls

The controls are independent:

| Control | Scope |
| --- | --- |
| `staleness.enabled` | Whether `scan`, `run`, `propose`, and `worker` execute for the selected config |
| `--dry-run` | Whether that invocation may persist or call agents/GitHub, where supported |
| `autoMerge` | Whether `propose` may request auto-merge after the evidence gate passes |
| `<ledger>/automerge.disabled` | Emergency merge kill switch; scanning and adjudication continue |
| `run.lock` | Prevents concurrent workers for one ledger |
| `state.lock` | Serializes mutable ledger transactions |

No one switch replaces the others. Disabling auto-merge does not disable
proposals, and disabling the canonical config does not disable a workflow
that selects a different config.

---

<a id="deployment-recipes"></a>

## Deployment Recipes

### Private Owner Only

- Commit `"enabled": false` in `paniolo.config.json`.
- Put `{"staleness":{"enabled":true}}` in the owner's ignored
  `paniolo.config.local.json`.
- Disable independent PR workflows and scheduled worker configurations.
- Run manual scans and workers from the owner's machine.

### Pull-Request Advisory Only

- Keep canonical automation disabled.
- Enable the dedicated read-only PR advisory config.
- Do not schedule `worker`; the advisory creates no ledger work.

### Fully Autonomous

- Enable canonical automation and configure role profiles and surfaces.
- Schedule `worker` on authorized local or self-hosted compute with signed-in
  agent and GitHub CLIs.
- Keep branch protection and `automerge.disabled` available as independent
  merge controls.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
