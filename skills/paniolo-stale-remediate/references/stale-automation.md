---
source-slug: stale-automation
source-hash: f2eca7c1374dd4be30b316f538deaca00a3a3a8eb134485fe7a9d1d005a8e283
bundled: 2026-09-28
title: Stale Automation
type: concept
tags:
- staleness
- harness-eng
- agents
- cli
updated: 2026-09-28
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
- [What Starts The Loop](#what-starts-the-loop)
- [Content Surfaces](#content-surfaces)
- [Declared Watches](#declared-watches)
- [Additional Candidate Lanes](#additional-candidate-lanes)
- [Historical And Versioned Claims](#historical-and-versioned-claims)
- [Allegation Lifecycle](#allegation-lifecycle)
- [Where The Code Lives](#where-the-code-lives)
- [See Also](#see-also)

---

<a id="the-loop"></a>

## The Loop

```text
declared watch, exact deleted reference, or admitted retrieval
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

<a id="what-starts-the-loop"></a>

## What Starts The Loop

The CLI does not schedule itself. Manual commands, a pull-request advisory,
or an external scheduler invoking `worker` start the loop. Each invocation
selects one configuration; an explicit workflow config is independent of the
canonical config and its machine-local overlay. See [stale-triggers](./stale-triggers.md) for the
trigger matrix and the exact boundary of `staleness.enabled`.

---

<a id="content-surfaces"></a>

## Content Surfaces

Three surfaces are measured independently; each has its own detector,
configured scope, and metrics:

| Surface | Detector | What it watches |
| --- | --- | --- |
| `wiki` | `declared-watch/1` | Registered wiki pages whose frontmatter declares watches |
| `docs` | `docs-declared/1` | Ordinary repository Markdown docs carrying `staleness:` frontmatter |
| `comment` | `comment-assoc/1` | Parser-owned code comments (tree-sitter; Rust, TypeScript/TSX/JS/JSX, C#, PowerShell, shell, and Python) bound to their owning symbol; Python docstrings are included |

The detector column names the declared-watch path. The exact-reference
audit can also nominate unwatched text on these surfaces. Volatile-claim
suggestions remain measurements, not allegations.

A deleted file marks open allegations at that path `obsolete`; generated
files and floating comments are skipped.

---

<a id="declared-watches"></a>

## Declared Watches

Pages opt in to the declared-watch lane through YAML frontmatter:

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

<a id="additional-candidate-lanes"></a>

## Additional Candidate Lanes

The `unwatched-ref/1` detector looks for exact references to deleted or
renamed code paths and symbols in wiki pages, ordinary docs, and bound code
comments. It does not require a watch. A location already covered by a
declared watch is suppressed here to avoid duplicate allegations. This is
an exact-reference audit, not a general semantic contradiction search.

The volatile audit emits capped suggestions for versions, dates, counts,
and moving phrases. A suggestion is not a stale verdict or an
allegation. Promotion requires a declared watch **and** source evidence
that the same literal changed from an old to a new value at the named
revision. Routine scans do not yet extract those old/new source literals,
so automatic volatile promotion remains off. See [stale-calibration](./stale-calibration.md)
for per-class measurements and the admission boundary.

---

<a id="historical-and-versioned-claims"></a>

## Historical And Versioned Claims

On Markdown pages, an exact marker pair exempts only the enclosed
historical passage from the deleted-reference audit:

```markdown
<!-- paniolo:historical:start -->
This passage records the old behavior.
<!-- paniolo:historical:end -->
```

The markers must stand on their own lines, pair exactly, and never nest.
Malformed ranges fail wiki validation or ordinary-doc collection. The
rest of the page and code comments remain eligible.

A watched section may declare its claim's intended scope in its watch or
with a section-local marker:

```markdown
## Earlier Release

<!-- paniolo:claim-scope version=paniolo-0.4.x as-of=2026-06 -->
```

The marker applies to that heading's watched passage, not the whole
page. A duplicate marker or a conflict with the watch is invalid. An
unmarked claim has unknown time/version scope; the verifier must not
infer currentness from age alone. When the named version resolves to a
Git revision, the evidence packet includes a bounded historical source
excerpt. Otherwise the agents must abstain if scope cannot be grounded.

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
  `paniolo stale` command surface; `stale_shadow.rs` holds the frozen
  `shadow-qmd` replay producer and the opt-in live qmd measurement lane.
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
