---
source-slug: stale-calibration
source-hash: 52cd0f512f39b23033dbba70493bcc2bffc28910e040eb202915fa0ec912e321
bundled: 2026-09-29
title: Stale Calibration
type: concept
tags:
- staleness
- harness-eng
- evals
updated: 2026-09-28
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
- [Next-Lane Scoring Safeguards](#next-lane-scoring-safeguards)
- [Canaries And The Dogfood Report](#canaries-and-the-dogfood-report)
- [See Also](#see-also)

---

<a id="the-corpus"></a>

## The Corpus

`crates/staleness/corpus/manifest.json` is the committed known-answer
corpus: a fingerprinted manifest (276 cases plus 72 retrieval distractors,
split seed `20260924`) partitioned into calibration and holdout splits.
Cases come from:

- **Seeded mutations** — `seeded.rs` generates fixtures by mutating real
  prose/code pairs, so gold labels are known by construction.
- **Mined history** — `mine.rs` extracts silver pairs from workspace git
  history.
- **Constructed pairs** — hand-built contradiction cases.

Gold provenance is restricted to `seeded-mutation`,
`deterministic-contradiction`, and `constructed-pair` — a case cannot claim
gold from retrieval scores or agent agreement. Splits stay honest: holdout
cases drive admission decisions; calibration cases tune.

Next-lane cases distinguish source invalidation, unwatched references,
watch coverage, temporal scope, volatile claims, and agent reports.
Non-baseline cases require a source revision so a current-code snapshot
cannot masquerade as the revision that a historical claim described.

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
   shadow-observation file standalone — no agents, no ledger. A missing
   or invalid index-integrity manifest fails scoring rather than adding
   a false retrieval miss.

Live dogfood collection is separate from the frozen holdout replay. Set
`staleness.retrieval.shadow.enabled: true` to make ordinary live `scan` and
`worker` runs record qmd's document fusion pool, the complete section pool
inside those documents, and the reranked section order. Each `R-` record
includes the normalized query, query-template and producer versions, exact
model references, ranks, scores, and the top K that would be scheduled.

The live lane remains non-authoritative: it creates no allegations and sends
nothing to the verifier. A candidate that overlaps a declared allegation can
later be joined to independently produced outcomes by changed-entity and
section IDs. A fuzzy-only candidate remains unknown, not negative, until a
future preregistered tail sampler or known-answer case labels it.

This uses one ledger rather than a second fuzzy ledger. Each live `R-` run
records a lexical-only document lane and a hybrid lexical-plus-vector document
lane for the same query. `paniolo stale report` compares them by canonical
document ID, exposing overlap, hybrid-only discoveries, and lexical-only
displacements. It also counts reranked sections and the top K that would have
been scheduled. Those differences measure retrieval behavior; they are not
recall or correctness claims until independent labels exist.

The qmd lane is **not admitted to production today**. Its producer is
implemented, but the earlier observation file has a superseded corpus
fingerprint. A fresh complete holdout replay must pass every fuzzy-lane gate
before production scan may use qmd nomination or reranking to create work.
Live shadow recording does not count as admission.

---

<a id="next-lane-scoring-safeguards"></a>

## Next-Lane Scoring Safeguards

Every scored qmd shadow run needs a run-scoped index manifest. It records
source revisions and hashes, document and chunk identities, and active
embedding fingerprints. The audit compares **all** eligible current
sources with **all** active index documents, not only a displayed sample.
Missing, stale, ghost, duplicate, orphaned, unembedded, or mixed-model
entries fail the shadow collection. Deterministic detection continues;
index lag is reported separately from a document-pool miss or a
section-rerank miss. An invalid run cannot improve or dilute Recall@K.

Lane scoring separates incremental gold recall, nomination volume,
precision, and a fixed below-top-K tail. Unlabeled tail items remain
unknown. Routing weights fit only on calibration lineages, then freeze
with a fingerprint and per-case top-K budget before holdout replay.
Zero-weight or omitted lanes cannot hide relevant sections they dropped
from the scheduled tail. A failed surface or tail gate keeps the lane
shadow-only; a passing score still changes no allegation verdict.

Volatile suggestions report separate version, date, count, and
moving-phrase totals. They are not false reports merely because no
outcome exists. Agent-filed `observed-in-use/1` reports also form a
separate shadow slice: only an exact claim-and-location match to an
independently verified stale or fresh outcome enters precision. Unknown
reports stay outside that denominator. See [stale-ledger](./stale-ledger.md) for `A-`
records and plan-staleness-next-lanes for remaining
operational gates.

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
automerge off), `automerge_recommended`, and the live lexical-versus-hybrid
retrieval comparison.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
