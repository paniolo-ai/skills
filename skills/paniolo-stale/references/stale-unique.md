---
source-slug: stale-unique
source-hash: 0062cf3e733b12ae1b719b8db4f8971ef39e360c51810a8c74601ea493e938ef
bundled: 2026-09-27
title: Why Stale Is Unique
type: synthesis
tags:
- staleness
- qmd
- harness-eng
- retrieval
updated: 2026-09-27
---

# Why Stale Is Unique

`paniolo stale` answers the question every agentic knowledge base quietly
fails: *is what the agent just retrieved still true?* Search solves
findability; staleness solves freshness. Together they are the loop a
knowledge base needs before agents can trust it — and freshness is the half
the industry keeps skipping.

## Contents

- [The Gap It Fills](#the-gap-it-fills)
- [What Makes It Different](#what-makes-it-different)
- [How It Combines With qmd](#how-it-combines-with-qmd)
- [The Compounding Asset](#the-compounding-asset)
- [See Also](#see-also)

---

<a id="the-gap-it-fills"></a>

## The Gap It Fills

An agentic knowledge base — LLM wiki, docs corpus, runbooks — is only as
good as its worst silent lie. Code moves; prose stays. Retrieval then does
its job perfectly and returns fluent, confidently wrong answers. The agent
has no way to know the page it found predates three refactorings.

The common responses all leak:

- **Lint/schema checks** prove a page is well-formed, not that it is true.
- **Freshness heuristics** (mtime, "last reviewed" stamps) measure time,
  not invalidation — an untouched page can be fine or false.
- **Regenerating docs wholesale** produces prose with no provenance,
  destroys the curation the wiki exists to hold, and still cannot say what
  changed semantically.
- **A model that "just checks"** with no evidence boundary grades its own
  homework: the same retrieval blind spot that served the page decides
  whether it is stale.

What was missing is a maintenance loop with the same rigor as the ingestion
loop: change detection, falsifiable claims, independent verification,
auditable dispositions. That is the stale ledger.

---

<a id="what-makes-it-different"></a>

## What Makes It Different

- **Detection nominates; it never proves.** A diff produces a falsifiable
  allegation about an exact location and claim — not a verdict. No score,
  agent agreement, or validator result can mint truth ([stale-automation](./stale-automation.md)).
- **Adjudication is adversarial by construction.** Verdicts are sealed
  before an independent challenger sees them; the challenger gets the
  verdict body, never the verifier's reasoning. A remediator's patch faces
  a separate patch challenger, and a merge gate — not a model — decides
  what lands ([stale-adjudication](./stale-adjudication.md)).
- **The ledger is durable, git-native state.** Content-addressed
  allegations, evidence, observations, and bundles live beside the prose
  they cover; cursors advance only when the ledger PR merges. Disagreement
  retains work instead of inventing resolution ([stale-ledger](./stale-ledger.md)).
- **Unknown is not negative.** Unverified pairs are excluded from
  denominators rather than counted as fresh — the system is built to not
  teach itself its own blind spots.
- **It runs on the customer's own agents.** Verification, challenge, and
  remediation execute through admitted CLI adapters on customer-controlled
  compute — no hosted black box deciding what your docs may say.
- **It is measured against known answers.** A committed corpus, frozen
  gates, planted canaries, and a false-resolution budget of zero make the
  pipeline falsifiable end to end ([stale-calibration](./stale-calibration.md)).

---

<a id="how-it-combines-with-qmd"></a>

## How It Combines With qmd

qmd is the recall layer; stale is the maintenance layer. qmd makes the
corpus findable — BM25, embeddings, reranking over pages and sections.
Stale keeps it true — and *uses* qmd to do it, carefully:

- **Retrieval as nomination, reranking as routing.** After a diff, each
  detection lane — declared watches, lexical search, vector search —
  produces a bounded candidate pool. The qmd reranker orders the pool's
  sections and schedules the top K per changed entity for verification. It
  is a routing stage only: a rerank score cannot create an allegation, set
  a disposition, corroborate a verdict, or feed merge confidence — and a
  low score defers a candidate to a durable queue rather than declaring the
  prose fresh.
- **Two separately measured stages.** Document recall (did retrieval find
  the page at all) and section recall (did reranking surface the right
  passage) are evaluated independently, because a reranker cannot recover
  a document absent from the pool. `stale shadow-qmd` measures both over
  the committed holdout corpus and feeds `replay --full`.
- **Shadow-first, never assumed.** Fuzzy lanes are shadow-only until a
  measured replay against the frozen gates admits them — the design even
  records how an earlier replay's oracle baseline was invalidated and had
  to be re-measured against the current corpus fingerprint. qmd earns its
  place in the pipeline with evidence.
- **Clean separation in storage.** The default ledger lives at
  `.paniolo/staleness`, deliberately outside the content roots, so
  candidates never enter the qmd corpus they protect.
- **No leakage to judges.** Verifier and challenger never see rerank
  scores, gold labels, or split assignments — the payload is
  redaction-checked for exactly those keys. Retrieval improves *which*
  prose gets reviewed; it never biases *whether* it is judged stale.

This is also the honest answer to "why not let the model decide": an
earlier code-comment drift spike rejected embeddings and the reranker as
truth judges. Asking retrieval to *nominate* instead is a better-shaped use
of the model — and it is the one that survived contact with measurement.

---

<a id="the-compounding-asset"></a>

## The Compounding Asset

Every run records the full funnel — candidates, ranks, verdicts, patches,
merge outcomes, and later reopenings — into the ledger's `R-`/`O-`/`B-`
records. Accumulated change-to-section rankings, verdicts, corrections, and
delayed outcomes are the compounding retrieval asset: they sharpen query
templates, reranker choice, top-K scheduling, and tail exploration over
time, and they are what a fresh install cannot download.

That is the business position too: audits deliver the wiki and harness with
a visible maintenance queue, and every resolved allegation is durable
evidence that the knowledge remains agent-ready. The commercial test is
operational — does the queue catch meaningful drift early, stay bounded,
and let inexpensive agents resolve the mechanical majority without routine
human work — and the ledger is built to answer it with numbers, not vibes.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
