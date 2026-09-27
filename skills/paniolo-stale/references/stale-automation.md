---
source-slug: stale-automation
source-hash: f2f894166b65422caf1f2d5395da72ef490355e44d33f9759276a63eeaa867cc
bundled: 2026-09-27
title: Stale Automation
type: concept
tags:
- staleness
- harness-eng
- agents
- cli
updated: 2026-09-27
---

# Stale Automation

`paniolo stale` is the staleness ledger: a durable, git-native queue of
falsifiable allegations that a code change may have invalidated prose. It
implements the code-invalidates-prose maintenance loop designed in
design-staleness-ledger and sequenced in
plan-staleness-automation.

Detection nominates bounded work; autonomous agents verify and remediate it;
repository gates record the outcome beside the fix. The core invariant —
**detection nominates but never proves truth** — is enforced in code: no
retrieval score, agent agreement, or validator result can mint a gold label
or set a disposition.

## Contents

- [The Loop](#the-loop)
- [Content Surfaces](#content-surfaces)
- [Declared Watches](#declared-watches)
- [Allegation Lifecycle](#allegation-lifecycle)
- [Where The Code Lives](#where-the-code-lives)
- [See Also](#see-also)

---

<a id="the-loop"></a>

## The Loop

```text
declared watch or admitted retrieval
  → allegation and immutable evidence
  → verifier verdict
  → independent verdict challenger
  → minimal remediation patch
  → independent patch challenger
  → repository gates / merge gate
  → automatic merge or insufficient-evidence retention
```

Each stage is a separate concern. Deterministic detectors turn a commit
range into candidate allegations; agent roles adjudicate them through
independent invocations; the merge gate decides what may land. A claim that
cannot be grounded is retained as `insufficient-evidence`, never silently
dropped.

---

<a id="content-surfaces"></a>

## Content Surfaces

Three surfaces are measured independently; each has its own detector,
configured scope, and metrics:

| Surface | Detector | What it watches |
| --- | --- | --- |
| `wiki` | `declared-watch/1` | Registered wiki pages whose frontmatter declares watches |
| `docs` | `docs-declared/1` | Ordinary repository Markdown docs carrying `staleness:` frontmatter |
| `comment` | `comment-assoc/1` | Parser-owned code comments (tree-sitter; Rust and TypeScript/TSX/JS/JSX) bound to their owning symbol |

A deleted file marks open allegations at that path `obsolete`; generated
files and floating comments are skipped.

---

<a id="declared-watches"></a>

## Declared Watches

Pages opt in through YAML frontmatter — nothing is inferred:

```yaml
staleness:
  mode: flag
  watches:
    - repo: ranch-hand
      paths: ["crates/auth-api/src/routes.rs"]
      symbols: ["unread_count"]
      section: "setup/windows"
```

- `mode: flag` is the only defined mode; it is reserved for future drafting
  modes.
- `repo` is a configured workspace repository key (required).
- `paths` are repo-relative; a trailing `/` means a directory prefix.
- `symbols` match a bare name and its qualified forms (`mod::name`).
- `section` is an optional slash-separated heading anchor; omitted means
  the whole file.

Breadth is bounded: more than 4 watches per page, or more than 8 paths or
8 symbols per watch, produces warnings. Validation rejects `unknown-repo`,
`empty-scope`, `escaping-path`, `invalid-symbol`, `invalid-section`, and
`unknown-mode` declarations.

---

<a id="allegation-lifecycle"></a>

## Allegation Lifecycle

| State | Meaning |
| --- | --- |
| `pending-verification` | No verifier has established a supported verdict |
| `confirmed-stale` | Verifier and challenger agree the claim is true |
| `remediation-proposed` | An isolated prose patch awaits gates |
| `resolved-updated` | The patch passed every required gate and landed |
| `dismissed` | Independent verification found the claim false |
| `insufficient-evidence` | Verifiers disagree or cannot ground a verdict; durable and re-enters verification on new evidence |
| `obsolete` | The prose location no longer exists |

The open queue is `pending-verification`, `confirmed-stale`,
`remediation-proposed`, and `insufficient-evidence`. A content edit returns
open *and resolved* work at the location to `pending-verification` — edits
never auto-resolve. See [stale-ledger](./stale-ledger.md) for the store layout and the full
transition table.

---

<a id="where-the-code-lives"></a>

## Where The Code Lives

- `ranch-hand/crates/staleness/` — the product-neutral engine: detection,
  the store, agent orchestration, remediation, merge gate, corpus, and eval.
- `ranch-hand/crates/paniolo-cli/src/commands/stale.rs` — the
  `paniolo stale` command surface; `stale_shadow.rs` holds `shadow-qmd`.
- The `stale` cargo feature gates the command and is part of
  `release-core`, so every release lane builds it. A binary without the
  feature reports `unrecognized subcommand 'stale'`.
- `ranch-hand/.github/workflows/staleness-advisory.yml` — the read-only PR
  advisory that runs `stale scan --dry-run` on `base..head`.

Operator-facing detail lives in [stale-commands](./stale-commands.md) (verbs and flags),
[stale-configuration](./stale-configuration.md) (`paniolo.config.json`), [stale-ledger](./stale-ledger.md) (state
and directories), [stale-adjudication](./stale-adjudication.md) (agent roles through merge), and
[stale-calibration](./stale-calibration.md) (corpus, gates, replay, report).

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
