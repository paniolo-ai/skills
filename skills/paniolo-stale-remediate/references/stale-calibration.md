---
source-slug: stale-calibration
source-hash: ad764cb20510ae1bc7035cf3c9784f499003208be2a350cdcec8b23bb65b7030
bundled: 2026-09-27
title: Stale Calibration
type: concept
tags:
- staleness
- harness-eng
- evals
updated: 2026-09-27
---

# Stale Calibration

The staleness engine is measured, not assumed. A committed known-answer
corpus, frozen gates, canary probes, and a deterministic dogfood report keep
detection and adjudication honest — and gate "fuzzy" retrieval lanes out of
production until they earn admission with measured recall.

## Contents

- [The Corpus](#the-corpus)
- [Labels](#labels)
- [Frozen Gates](#frozen-gates)
- [Replay And Shadow Lanes](#replay-and-shadow-lanes)
- [Canaries And The Dogfood Report](#canaries-and-the-dogfood-report)
- [See Also](#see-also)

---

<a id="the-corpus"></a>

## The Corpus

`crates/staleness/corpus/manifest.json` is the committed known-answer
corpus: a fingerprinted manifest (~100+ cases, split seed `20260924`)
partitioned into calibration and holdout splits. Cases come from:

- **Seeded mutations** — `seeded.rs` generates fixtures by mutating real
  prose/code pairs, so gold labels are known by construction.
- **Mined history** — `mine.rs` extracts silver pairs from workspace git
  history.
- **Constructed pairs** — hand-built contradiction cases.

Gold provenance is restricted to `seeded-mutation`,
`deterministic-contradiction`, and `constructed-pair` — a case cannot claim
gold from retrieval scores or agent agreement. Splits stay honest: holdout
cases drive admission decisions; calibration cases tune.

---

<a id="labels"></a>

## Labels

Labels (`L-` records) are the three-axis schema from
design-staleness-ledger: surface, verdict value, and
provenance/strength. They are immutable — a correction is a new label
naming the record it supersedes. Critically, **unlabeled pairs are unknown,
not negative**, and are excluded from denominators rather than counted as
failures.

---

<a id="frozen-gates"></a>

## Frozen Gates

`gates.rs` holds thresholds that do not move between runs:

| Gate | Threshold |
| --- | --- |
| seeded-watch recall | 1.0 |
| historical mechanical recall | 0.80 |
| nomination precision | 0.70 |
| verdict accuracy | 0.95 |
| critical errors | 0 (all kinds) |

Compute budgets bound each evaluation: 3 attempts per role, 8 invocations
per allegation, 12k packet tokens, 120 s per invocation, 15 minutes per
allegation.

Fuzzy (retrieval-based) lanes stay out of production until the admission
gates pass: minimum incremental recall 0.10, pool size 20–50, pool recall
0.95, top-K=2 recall 0.90, and passage reduction 0.50. Gate verdicts per
surface are `go`, `revise`, or `stop`.

---

<a id="replay-and-shadow-lanes"></a>

## Replay And Shadow Lanes

`paniolo stale replay` replays built-in known-answer cases through the
**real** adapters — preflight first (`<cli> --version`, 15 s per role) —
then evaluates the frozen gates. It writes ledger records, so point
`--root` at a disposable directory, never a live ledger.

The shadow lane measures candidate retrieval without touching production:

1. `paniolo stale shadow-qmd <output.json>` (requires the `qmd` feature)
   materializes `stale-shadow-*` collections in an isolated temp index over
   the committed docs holdout, retrieves a 20-document pool per case,
   reranks, and writes a `ShadowReplayInput` file.
2. `paniolo stale replay --full --shadow-observations <file>` scores the
   fuzzy lane's admission gates from that file. Without the file, `--full`
   reports no fuzzy-lane admission rather than fabricating a measurement.
3. `paniolo stale score-shadow <file>` validates and re-scores a
   shadow-observation file standalone — no agents, no ledger.

---

<a id="canaries-and-the-dogfood-report"></a>

## Canaries And The Dogfood Report

`paniolo stale seed <target> <claim> [--expected stale|fresh] [--source
repo:path] [--commit <sha>]` plants a known-answer allegation
(`canary/stale/2` or `canary/fresh/2`, severity `known-answer`) that flows
through normal adjudication — the known answer turns a live verdict into a
measurement. Seeds are idempotent because ids are content-addressed.

`paniolo stale report` recomputes the deterministic per-surface dogfood
report from the ledger: outcomes, sealed observations by role, conflicts, a
10% tail sample, canary hits and misses, the false-resolution error budget
(`MAX_FALSE_RESOLUTIONS = 0` — one false resolution recommends keeping
automerge off), and `automerge_recommended`.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
