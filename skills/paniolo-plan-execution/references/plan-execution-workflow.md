---
source-slug: plan-execution-workflow
source-hash: 1f4296cedf0079c1f14f17b8f2ef044503cb62997b0aa217ab932d9e0e8562ed
bundled: 2026-09-20
title: Plan Execution Workflow
type: concept
status: active
tags:
- authoring
- plans
- testing
- verification
updated: 2026-08-20
---

# Plan Execution Workflow

Executing a plan starts by auditing its premises against the current repository. Plans age, status
claims can be wrong, and partial implementations often leave disconnected consumers or untested
failure paths.

## Contents

- [Workflow](#workflow)
- [Evidence Output](#evidence-output)
- [Stop Conditions](#stop-conditions)
- [Completion Boundary](#completion-boundary)
- [See Also](#see-also)

---

<a id="workflow"></a>

## Workflow

1. Discover the plan, linked contracts, relevant skills, implementation symbols, and tests.
1. Inspect the dirty worktree and preserve unrelated changes.
1. Build one evidence row per card: contract, implementation paths, behavior test, observed result.
1. Reclassify each card as satisfied, defective, blocked by a dependency, or deferred.
1. Implement in dependency order and validate the smallest affected layer while iterating.
1. Trace cross-layer data from producer through validation, persistence, loading, state, and every
   consumer.
1. Diagnose end-to-end failures as runner, fixture, product, or cleanup defects before patching.
1. Run every release gate named by the plan; repeat shared-state gates when repeatability matters.
1. Update plan status and evidence from observed results, not attempts.
1. Turn repeated mistakes into durable rules, skills, concept pages, tests, or deterministic checks.

---

<a id="evidence-output"></a>

## Evidence Output

Maintain a card table during execution:

| Card | Contract | Implementation | Behavior Test | Result |
| --- | --- | --- | --- | --- |
| Card ID | Observable target | Paths or symbols | Test and assertion | Observed status |

Maintain a separate validation table:

| Gate | Environment | Result | Evidence Or Failure |
| --- | --- | --- | --- |
| Command or manual check | Supported host | Passed, failed, inconclusive, or not run | Output summary |

List remaining work, explicit deferrals, cleanup evidence, and unauthorized actions not taken.

---

<a id="stop-conditions"></a>

## Stop Conditions

Stop and request direction when a missing choice changes the product contract, completion requires
production mutation or destructive work outside the user's authority, or repository contracts
contradict one another. Continue through ordinary test failures, stale plan status, and local
implementation uncertainty when read-only evidence can resolve them.

Do not patch product validation for an invalid fixture, broaden process termination to clear a
port, or replace a required broad gate with a focused test.

---

<a id="completion-boundary"></a>

## Completion Boundary

Execution is complete when every in-scope card has behavior evidence, every required gate has an
honest result, temporary processes and test data are cleaned up, documentation reflects the landed
state, and deferred work remains visible.

An outer timeout is inconclusive unless an assertion failed first. Focused coverage cannot stand in
for an unrun broad suite.

---

<a id="see-also"></a>

## See Also

- Cross-Layer Data Flow Verification
- Plan Authoring Best Practices
- Plan Acceptance Evidence
- E2E Fixture Contract Validation
- Playwright Failure Classification
- Test-Owned Entities
