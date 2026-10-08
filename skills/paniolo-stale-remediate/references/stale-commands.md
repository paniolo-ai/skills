---
source-slug: stale-commands
source-hash: 997bfda4c423f4258b3326e6b26afbdfefcbd8585b863fdf0680f2c525cfa837
bundled: 2026-10-08
title: Stale Commands
type: concept
tags:
- staleness
- harness-eng
- cli
updated: 2026-10-06
---

# Stale Commands

The `paniolo stale` command family operates the staleness ledger described
in [stale-automation](./stale-automation.md). Every subcommand prints JSON to stdout and exits
zero on success; failures are errors on stderr with a nonzero exit. A held
worker `run.lock` is not an error — the worker prints
`{"stopped": "run lock held"}` and exits 0.

The command is gated behind the `stale` cargo feature (part of
`release-core`). Binaries built without it report
`unrecognized subcommand 'stale'`. Individual verbs marked *(qmd)* in
this reference exist only in builds that also link the `qmd` feature —
a `release-core` binary reports `unrecognized subcommand` for them.

## Contents

- [Global Flags](#global-flags)
- [Automation Gate](#automation-gate)
- [Inspection Verbs](#inspection-verbs)
- [Detection Verb](#detection-verb)
- [Agent Filing Verb](#agent-filing-verb)
- [Queue-Management Verbs](#queue-management-verbs)
- [Agent And Publication Verbs](#agent-and-publication-verbs)
- [Ledger Index And Verify Verbs](#ledger-index-and-verify-verbs)
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

`flag` has a stricter gate: the effective selected config must explicitly
set `staleness.enabled: true`. A missing section or an implicit default
does not authorize agent filing. Disabled filing returns `status: disabled`
and writes no agent-report record.

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
| `list` | `--actionable` | List allegations as `{id, state, section_id, claim, revision}` rows; `--actionable` keeps `pending-verification`, `confirmed-stale`, `remediation-proposed` — `insufficient-evidence` is retained open work and re-enters via `retry`/`worker --retry-retained` |
| `next` | — | Print the first actionable allegation in deterministic id order |
| `show <id>` | `S-` id or unique prefix | Show one allegation plus its sealed observations by role; an ambiguous prefix is an error |
| `report` | — | Deterministic per-surface outcomes, canaries, false-resolution budget, `automerge_recommended`, retrieval shadow, and separate agent-report slice |
| `score-shadow` | `<input>` JSON file | Score shadow observations only when their qmd index-integrity manifest is present and valid; no agents or ledger writes |

---

<a id="detection-verb"></a>

## Detection Verb

| Command | Arguments and flags | Behavior |
| --- | --- | --- |
| `scan` | `--code key:path` (repeatable), `--wiki key:path` (repeatable), `--base <sha>`, `--head <sha>`, `--sweep`, `--only <selector>` (repeatable), `--dry-run` | Detect allegations over `base..head` — or, under `--sweep`, over every in-scope comment and document on each `--code`; report watch coverage, scan errors, and volatile suggestions; `--only` bounds which locations file (out-of-scope matches count as `suppressed_out_of_scope`); a normal run writes allegation and retrieval-run records, while `--dry-run` touches no ledger |

---

<a id="agent-filing-verb"></a>

## Agent Filing Verb

| Command | Arguments and flags | Behavior |
| --- | --- | --- |
| `flag` | `--target <repo:path#anchor> --claim <text> --observation <file> [--source-revision <revision>]` | File one bounded observation the working agent already encountered. An identical filing returns `duplicate`; otherwise it returns `filed` and an `A-` report id. It does not investigate, verify, or repair the claim |

The observation argument names a UTF-8 file, not inline text. The closed
packet rejects patch requests, verdicts, and task-output fields. Only an
explicitly enabled config permits a write. See [stale-ledger](./stale-ledger.md) for the
record and [stale-calibration](./stale-calibration.md) for its independent-outcome measurement.

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
| `migrate` | `--apply` | Report the JSON→`workspace.sqlite` cutover plan — record counts per family, the resolved destination, whether it already carries a stale schema. Takes the run lock and writes nothing until `--apply` |

---

<a id="agent-and-publication-verbs"></a>

## Agent And Publication Verbs

| Command | Arguments and flags | Behavior |
| --- | --- | --- |
| `run` | `--adapter`, `--model`, `--challenger`, `--challenger-model`, `--remediator`, `--remediator-model`, `--patch-challenger`, `--patch-challenger-model`, `--repo key:path` (repeatable), `--only <selector>` (repeatable), `--max`, `--timeout-ms`, `--max-output-bytes` | Verify `pending-verification` allegations (verifier + verdict challenger), then remediate `confirmed-stale` ones (remediator + patch challenger). All role and limit flags are one-run overrides of config; `--only` restricts both phases to matching allegation locations |
| `propose` | `--repo key:path` (repeatable), `--only <selector>` (repeatable), `--dry-run` | Push each remote's in-scope `remediation-proposed` allegations as one PR on a `staleness/rem-*` branch via `gh pr create`, record the pending proposal, and request `gh pr merge --auto --squash --delete-branch` when `autoMerge` is set and the merge gate passes; out-of-scope allegations stay `remediation-proposed` and report `skipped: out-of-scope` |
| `merge-sync` | `--repo key:path` (repeatable), `--only <selector>` (repeatable), `--dry-run` | Reconcile pending proposals with `gh pr list`: merged heads run the merge gate and resolve the group; closed PRs clear the proposal for re-queue; `--only` reconciles a proposal only when every record it carries is in scope |
| `worker` | `--ledger-repo key:path`, `--repo key:path` (repeatable), `--wiki key:path` (repeatable), the `run` role/limit flags, `--bootstrap`, `--sweep`, `--retry-retained`, `--only <selector>` (repeatable), `--dry-run` | The durable runner: merge-sync → checkpoint-gated scan per repo → adjudicate → propose → publish the ledger itself as a candidate PR on branch `staleness/ledger` → maintenance tail: claim merge-minted `verify` jobs against the cycle's checkout map and reproject the ledger into the qmd `stale-ledger` collection. One run at a time via `run.lock`; one `--only` set bounds filing, adjudication, remediation, and proposal while checkpoints still advance for every `--repo` |

`run` and `worker` admit `codex`, `claude`, `cursor`, and `devin`; any
other name is rejected before work starts. `--bootstrap` records each
`--repo`'s HEAD as its scan checkpoint and skips scanning history — use it
on first contact. `--retry-retained` requeues `insufficient-evidence` allegations
before adjudication. `propose`, `merge-sync`, and `worker` require `git`
and `gh` on PATH with GitHub authentication.

The `worker` maintenance tail needs the `qmd` feature: with it, a
published cycle claims the `verify` jobs `merge-sync` minted (lease
identity `stale-worker`, the same checkout map the cycle assembled) and
reprojects the ledger's search documents — see the
[maintenance verbs](#ledger-index-and-verify-verbs). Without `qmd` the
report carries `{ "skipped": "qmd engine not in this build" }` under
`verify` and `index`. The tail is best-effort and skipped on `--dry-run`
or cancellation: a qmd-side failure lands as an `error` marker in the
report, never as a failure of an already-published cycle.

---

<a id="ledger-index-and-verify-verbs"></a>

## Ledger Index And Verify Verbs

These verbs all require the `qmd` feature — they read and write the
shared `workspace.sqlite` index through `IndexStore` and never embed.
Standalone invocations stay useful for manual refresh or repair; the
scheduled `worker` tail already covers the routine cases.

| Command | Arguments and flags | Behavior |
| --- | --- | --- |
| `index` *(qmd)* | — | Reproject the ledger into the shared database's reserved `stale-ledger` qmd collection — one bounded search document per allegation. Scoped queries can opt in to the collection; unscoped search and prompt hooks never return ledger rows |
| `verify` *(qmd)* | `--repo key:path` (repeatable), `--worker <name>` (default `stale-verify`), `--limit <n>` (default 8) | Claim queued `verify` jobs minted by merged remediations; a checkout that does not contain the merge head releases the job rather than confirming closure; a matching checkout refreshes the indexed passage to its bytes and completes the job |
| `packet <id>` *(qmd)* | `S-` id or unique prefix, `--repo key:path` (repeatable), `--budget <bytes>` | Emit the bounded DB20 remediation packet for one allegation — passage bytes at the checkout's HEAD, freshness- and ledger-filtered qmd neighbors, and prior fixes. Read-only |

---

<a id="calibration-verbs"></a>

## Calibration Verbs

| Command | Arguments and flags | Behavior |
| --- | --- | --- |
| `replay` | `--adapter`, `--model`, `--challenger-adapter`, `--challenger-model`, `--full`, `--shadow-observations <file>` (requires `--full`) | Live known-answer replay through the real adapters: preflight each role (`<cli> --version`, 15 s), then run built-in cases and the frozen gate report. `--full` adds remediation cases and scores fuzzy-lane admission from a shadow-observation file. **Writes ledger records — use a disposable `--root`** |
| `shadow-qmd` | positional `<output>` JSON path, `--work-dir <dir>` | (requires the `qmd` feature) Measure qmd retrieval and reranking over the committed docs holdout in an isolated index; write observations with a run-scoped index-integrity manifest. Needs a configured qmd rerank model |
| `seed` | `<target> <claim>`, `--expected stale\|fresh` (default `stale`), `--source repo:path`, `--commit` (default `HEAD`) | Plant a known-answer canary allegation (`canary/stale/2` or `canary/fresh/2`, severity `known-answer`) that flows through normal adjudication so the report can measure hits and misses |
| `eval` *(qmd)* | `--cases <file>`, `--limit <n>` (default 10), `--db <path>` | Replay a frozen gold-labeled case file against the `stale-ledger` projection and report recall; report only — a corpus whose queries embed the gold labels fails the leakage check rather than scoring |

---

<a id="common-conventions"></a>

## Common Conventions

- `key:path` addresses a checkout: the key is a configured workspace
  repository name, the path is where it is checked out on disk.
- `--only <key[:glob[#anchor]]>` repeats on `scan`, `run`, `propose`,
  `merge-sync`, and `worker` to bound which locations that invocation
  files, verifies, remediates, or proposes. A bare key selects the whole
  repository; `*` matches direct children, `**` any depth, and `#anchor`
  qualifies an exact path. Selectors union; an empty match set is zero
  work, never a repository-wide fallback. Scan checkpoints still advance
  for every `--repo` — the selector bounds work, not coverage
  bookkeeping — and `run.lock`, `staleness.enabled`, and
  `automerge.disabled` keep their unscoped meaning.
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
