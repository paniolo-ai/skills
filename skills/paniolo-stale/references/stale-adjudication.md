---
source-slug: stale-adjudication
source-hash: dbb1ad3818d42859cfd31f812f08b2d2bd019973a0221a8a08120c61971c874f
bundled: 2026-09-27
title: Stale Adjudication
type: concept
tags:
- staleness
- harness-eng
- agents
updated: 2026-09-27
---

# Stale Adjudication

Adjudication is the agent half of the staleness loop: fresh, isolated
coding-agent subprocesses verify allegations, challenge verdicts, author
remediation patches, and challenge those patches — while the Rust engine
keeps every state transition, seal, and merge decision deterministic. Per
decision-staleness-rust-cli-agents, the engine owns the
ledger; the CLIs are interchangeable evidence producers.

## Contents

- [Roles](#roles)
- [Adapters](#adapters)
- [Verdict Dispositions](#verdict-dispositions)
- [Remediation Patches](#remediation-patches)
- [The Merge Gate](#the-merge-gate)
- [Proposals And The Worker](#proposals-and-the-worker)
- [CI Advisory](#ci-advisory)
- [Operational Gotchas](#operational-gotchas)
- [See Also](#see-also)

---

<a id="roles"></a>

## Roles

Four independently invoked roles, each a fresh process with no resume state
or shared conversation:

| Role | Job |
| --- | --- |
| verifier | Judge the allegation against bounded evidence; emit a strict verdict |
| verdictChallenger | Independently attack the sealed verdict — it receives the verdict body, never the verifier's reasoning |
| remediator | Author a minimal unified-diff patch confined to the alleged span |
| patchChallenger | Independently test the remediator's patch |

Config assigns each role an agent profile; CLI flags override per run
([stale-configuration](./stale-configuration.md)). One bounded repair retry is allowed per
invocation; after that the phase reports the failure instead of guessing.

---

<a id="adapters"></a>

## Adapters

An adapter is a conformance-approved `ProcessSpec` — a fixed CLI
invocation, not a shell template. Admitted adapters: `codex`, `claude`,
`cursor`. A `devin` spec exists in code but is not admitted by the CLI.

| Adapter | Invocation shape |
| --- | --- |
| `codex` | `codex exec --sandbox read-only --skip-git-repo-check --json --model <model> -` — payload on stdin, last `agent_message.text` JSONL entry is the response |
| `claude` | `claude -p --output-format json --model <model> --disallowedTools Bash,Write,Edit,NotebookEdit,Read,WebFetch,WebSearch` — stdin payload, `.result` envelope |
| `cursor` | `cursor-agent -p --output-format json --mode ask --model <model> <input>` — payload as argv, `.result` envelope |

Every invocation runs with a scrubbed environment: `env_clear` plus an
allowlist (`PATH`, `HOME`, `USERPROFILE`, `XDG_CONFIG_HOME`, and
`SYSTEMROOT`/`COMSPEC`/`TEMP`/`TMP` on Windows, `TMPDIR` elsewhere).
**No secrets propagate** — each CLI authenticates through its own signed-in
config under `HOME`. Stdout is capped at `maxOutputBytes`; the process tree
is killed after `timeoutMs`. On Windows, `.cmd`/`.bat` shims resolve through
`cmd /c` or direct node invocation and `.ps1` through PowerShell.

The payload is `{instructions, request}` — role instructions plus the
serialized evidence request — and is redaction-checked so forbidden keys
(`score`, `gold`, `label`, `labels`, `expected`, `rerank`, `hidden`,
`split`) can never leak an answer to a judge.

---

<a id="verdict-dispositions"></a>

## Verdict Dispositions

The verifier's response is strict-parsed (unknown fields rejected) and
validated: a `relevant` finding requires a verdict; `stale`/`fresh` verdicts
need citations that resolve to `X-` excerpt ids with matching content hashes
inside `permitted_repos`. The observation is **sealed immutable before the
challenger runs**, so the challenge targets exactly what was produced.

| Verdict | Challenger | Disposition |
| --- | --- | --- |
| relevant + stale | upheld | `confirmed-stale` |
| relevant + fresh, or irrelevant | upheld | `dismissed` |
| any | not upheld | `insufficient-evidence` |
| adapter error / unparseable | — | retry once, else `insufficient-evidence` — never a disposition |

Re-running an allegation already in its adjudicated state records no
transition — adjudication is idempotent.

---

<a id="remediation-patches"></a>

## Remediation Patches

For `confirmed-stale` work the remediator emits a unified diff constrained
to exactly the alleged file and section span: no file creates, deletes,
renames, or mode changes; frontmatter is untouchable; context lines must
match exactly. The engine parses and applies the patch in memory — through
its own parser, not `git apply` — and the patch challenger independently
tests it. Success seals a `B-` remediation bundle and moves the allegation
to `remediation-proposed`. Code-comment allegations take a separate
comment-remediation path.

Sealed patches can go stale: at apply time, if the target's content hash no
longer matches the bundle, or hunks no longer apply, the allegation is
requeued to `pending-verification` rather than force-applied or failed.

---

<a id="the-merge-gate"></a>

## The Merge Gate

`merge_gate` authorizes a merge only when all of these hold:

- The `<ledger>/automerge.disabled` kill-switch file is absent.
- The allegation is `remediation-proposed`.
- A pending proposal exists whose recorded head SHA matches the merge head
  exactly.
- A cycle-bound `B-` bundle exists whose revision plus one is current, with
  verifier and remediator observations present, both challengers `upheld`,
  matching cycle ids and parent links, and a matching patch hash.

Groups apply all-or-nothing. With `autoMerge` configured, `propose`
requests `gh pr merge --auto --squash --delete-branch` only after this gate
authorizes the exact proposed head.

---

<a id="proposals-and-the-worker"></a>

## Proposals And The Worker

`propose` turns `remediation-proposed` allegations into one PR per remote
on a `staleness/rem-*` branch (see [stale-ledger](./stale-ledger.md) for the worktree
mechanics) and records a pending proposal. `merge-sync` reconciles against
GitHub: `MERGED` heads run the merge gate and resolve the group; `CLOSED`
PRs clear the proposal so work re-queues.

`worker` is the durable runner — merge-sync → cursor-gated scan →
adjudicate → propose → publish the ledger itself as a PR on
`staleness/ledger`. One run per ledger via `run.lock`; a competing run
prints `{"stopped": "run lock held"}` and exits 0.

---

<a id="ci-advisory"></a>

## CI Advisory

`ranch-hand/.github/workflows/staleness-advisory.yml` runs on every pull
request with `contents: read` and no secrets — fork-safe by construction.
It checks out full history plus the `paniolo-wiki` repo into
`.staleness/paniolo-wiki`, builds `paniolo-cli` with only the `stale`
feature, then runs
`paniolo stale --root . --config .github/staleness-advisory.json scan
--code ranch-hand:. --wiki paniolo-wiki:.staleness/paniolo-wiki
--base <base> --head <head> --dry-run`.
The scan is `continue-on-error` and summarized into `$GITHUB_STEP_SUMMARY`
(capped at 8192 bytes) with the JSON uploaded as the `staleness-report`
artifact. It can be inconclusive; it can never block a merge.

---

<a id="operational-gotchas"></a>

## Operational Gotchas

- `replay` writes real ledger records — always run it against a disposable
  `--root`, never a live ledger.
- `propose`, `merge-sync`, and `worker` need `git` and an authenticated
  `gh` on PATH.
- Agent CLIs must be installed and signed in; preflight is
  `<cli> --version` with a 15-second timeout, and a failed preflight stops
  the phase without mutating the queue.
- Agent invocations get no inherited environment — anything a role needs
  must come from the adapter's own CLI config, not exported secrets.
- Evidence recorded without a `source_path` cannot authorize action;
  legacy records fail closed at packet time.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
