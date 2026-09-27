---
source-slug: stale-commands
source-hash: 84abb4ff5277aa7461f2fd99bb4f1767aebd0a85646ad0d10f11dfceb8d20227
bundled: 2026-09-27
title: Stale Commands
type: concept
tags:
- staleness
- harness-eng
- cli
updated: 2026-09-27
---

# Stale Commands

The `paniolo stale` command family operates the staleness ledger described
in [stale-automation](./stale-automation.md). Every subcommand prints JSON to stdout and exits
zero on success; failures are errors on stderr with a nonzero exit. A held
worker `run.lock` is not an error — the worker prints
`{"stopped": "run lock held"}` and exits 0.

The command is gated behind the `stale` cargo feature (part of
`release-core`). Binaries built without it report
`unrecognized subcommand 'stale'`.

## Contents

- [Global Flags](#global-flags)
- [Automation Gate](#automation-gate)
- [Inspection Verbs](#inspection-verbs)
- [Detection Verb](#detection-verb)
- [Queue-Management Verbs](#queue-management-verbs)
- [Agent And Publication Verbs](#agent-and-publication-verbs)
- [Calibration Verbs](#calibration-verbs)
- [Common Conventions](#common-conventions)
- [See Also](#see-also)

---

<a id="global-flags"></a>

## Global Flags

| Flag | Default | Meaning |
| --- | --- | --- |
| `--root <PATH>` | `.` | Repository root that hosts the ledger and the default config |
| `--config <PATH>` | `paniolo.config.json` under `--root` | Config file whose `staleness` section is read; see [stale-configuration](./stale-configuration.md) |

`--root` is canonicalized before use. A relative `--config` resolves under
`--root`; a missing explicit file is an error, while a missing default file
falls back to built-in defaults. `--config` must name a
`paniolo.config.json`-shaped document — the command reads its `staleness`
key, so a file holding only the bare block is treated as no section.

---

<a id="automation-gate"></a>

## Automation Gate

`staleness.enabled: false` disables `scan`, `run`, `propose`, and
`worker` for the selected config. Each returns exit zero with
`{"status":"disabled",...}` before opening the ledger. That response
means **not run**, not an empty scan or completed cycle.

Other commands remain available. A command using a differently named explicit
`--config` has independent policy and does not inherit the canonical config
or its machine-local overlay. See [stale-triggers](./stale-triggers.md) for the full trigger and
configuration-authority matrix.

---

<a id="inspection-verbs"></a>

## Inspection Verbs

These write nothing to the ledger.

| Command | Arguments and flags | Behavior |
| --- | --- | --- |
| `list` | `--actionable` | List allegations as `{id, state, section_id, claim, revision}` rows; `--actionable` keeps `pending-verification`, `confirmed-stale`, `remediation-proposed` |
| `next` | — | Print the first actionable allegation in deterministic id order |
| `show <id>` | `S-` id or unique prefix | Show one allegation plus its sealed observations by role; an ambiguous prefix is an error |
| `report` | — | Deterministic per-surface dogfood report: outcomes, sealed observations by role, canary hits/misses, false-resolution budget, `automerge_recommended` |
| `score-shadow` | `<input>` JSON file | Validate and score a complete shadow-observation file without running verifier or remediation agents |

---

<a id="detection-verb"></a>

## Detection Verb

| Command | Arguments and flags | Behavior |
| --- | --- | --- |
| `scan` | `--code key:path` (repeatable), `--wiki key:path` (repeatable), `--base <sha>`, `--head <sha>`, `--dry-run` | Detect allegations over `base..head`; a normal run writes allegation and retrieval-run records, while `--dry-run` computes the report without touching the ledger |

---

<a id="queue-management-verbs"></a>

## Queue-Management Verbs

These mutate allegation records only — no agent invocations, no git or
GitHub side effects.

| Command | Arguments and flags | Behavior |
| --- | --- | --- |
| `retry <id>` | `S-` id or unique prefix | Requeue `insufficient-evidence`, `dismissed`, or `resolved-updated` work to `pending-verification` |
| `resolve <id>` | `--note <text>` (required) | Mark a `remediation-proposed` allegation `resolved-updated` when its fix landed outside a staleness PR; the note records the carrying commit or PR |
| `prune` | `--apply` | Remove terminal (`resolved-updated`, `obsolete`) allegations; without `--apply` prints the plan only |
| `rebase` | `--apply` | Re-resolve open locations; a missing target file becomes `obsolete`. Plan-only without `--apply` |

---

<a id="agent-and-publication-verbs"></a>

## Agent And Publication Verbs

| Command | Arguments and flags | Behavior |
| --- | --- | --- |
| `run` | `--adapter`, `--model`, `--challenger`, `--challenger-model`, `--remediator`, `--remediator-model`, `--patch-challenger`, `--patch-challenger-model`, `--repo key:path` (repeatable), `--max`, `--timeout-ms`, `--max-output-bytes` | Verify `pending-verification` allegations (verifier + verdict challenger), then remediate `confirmed-stale` ones (remediator + patch challenger). All role and limit flags are one-run overrides of config |
| `propose` | `--repo key:path` (repeatable), `--dry-run` | Push each remote's `remediation-proposed` allegations as one PR on a `staleness/rem-*` branch via `gh pr create`, record the pending proposal, and request `gh pr merge --auto --squash --delete-branch` when `autoMerge` is set and the merge gate passes |
| `merge-sync` | `--repo key:path` (repeatable), `--dry-run` | Reconcile pending proposals with `gh pr list`: merged heads run the merge gate and resolve the group; closed PRs clear the proposal for re-queue |
| `worker` | `--ledger-repo key:path`, `--repo key:path` (repeatable), `--wiki key:path` (repeatable), the `run` role/limit flags, `--bootstrap`, `--retry-retained`, `--dry-run` | The durable runner: merge-sync → cursor-gated scan per repo → adjudicate → propose → publish the ledger itself as a candidate PR on branch `staleness/ledger`. One run at a time via `run.lock` |

`run` and `worker` admit the adapters `codex`, `claude`, and `cursor`; any
other name is rejected before work starts. `--bootstrap` records each
`--repo`'s HEAD as its cursor and skips scanning history — use it on first
contact. `--retry-retained` requeues `insufficient-evidence` allegations
before adjudication. `propose`, `merge-sync`, and `worker` require `git`
and `gh` on PATH with GitHub authentication.

---

<a id="calibration-verbs"></a>

## Calibration Verbs

| Command | Arguments and flags | Behavior |
| --- | --- | --- |
| `replay` | `--adapter`, `--model`, `--challenger-adapter`, `--challenger-model`, `--full`, `--shadow-observations <file>` (requires `--full`) | Live known-answer replay through the real adapters: preflight each role (`<cli> --version`, 15 s), then run built-in cases and the frozen gate report. `--full` adds remediation cases and scores fuzzy-lane admission from a shadow-observation file. **Writes ledger records — use a disposable `--root`** |
| `shadow-qmd` | positional `<output>` JSON path, `--work-dir <dir>` | (requires the `qmd` feature) Measure qmd retrieval and reranking over the committed docs holdout: materialize `stale-shadow-*` collections in an isolated index, retrieve and rerank per case, write the `ShadowReplayInput` that `replay --full` consumes. Needs a configured qmd rerank model |
| `seed` | `<target> <claim>`, `--expected stale\|fresh` (default `stale`), `--source repo:path`, `--commit` (default `HEAD`) | Plant a known-answer canary allegation (`canary/stale/2` or `canary/fresh/2`, severity `known-answer`) that flows through normal adjudication so the report can measure hits and misses |

---

<a id="common-conventions"></a>

## Common Conventions

- `key:path` addresses a checkout: the key is a configured workspace
  repository name, the path is where it is checked out on disk.
- Allegation ids print as `S-<hash>`; a unique prefix is accepted anywhere
  an id is.
- `--dry-run` on `scan`, `propose`, `merge-sync`, and `worker` reports the
  plan without writing ledger records, touching git, or calling agents.
- `prune` and `rebase` are plan-only until `--apply` is present.
- `shadow-qmd` writes its requested JSON output and temporary index artifacts,
  but it does not open the ledger or invoke adjudication agents.
- Preflight failure inside `run` is not an error: the report carries
  `stopped: "preflight <role>: …"` and the queue is untouched.
- Evidence content is read at the recorded commit via `git show`; `--repo`
  checkouts must be readable for prose and evidence excerpts.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
