---
name: paniolo-plan
description: |
  Author, execute, or audit implementation plans with dependency-ordered cards, observable contracts, test-backed acceptance criteria, and honest verification gates. Use when creating or refining a plan before implementation begins, or when implementing and auditing an approved plan.
license: MIT
metadata:
  version: 0.2.0
tags:
- plans
- authoring
- implementation
- verification
user-invocable: true
references:
- references/completion-means-the-gate-ran.md
- references/cross-layer-data-flow-verification.md
- references/plan-acceptance-evidence.md
- references/plan-authoring-best-practices.md
- references/plan-execution-workflow.md
- references/playwright-e2e-fixture-contract-validation.md
- references/playwright-failure-classification.md
- references/playwright-test-owned-entities.md
---

**Requires:** file-read, repository search, file-write, and terminal access for verification gates.

# Implementation Plan Authoring and Execution

## Preconditions

- Read repository instructions, existing plans, linked contracts, and dirty worktree status.
- Inspect current code and tests before confirming a defect or claiming completion.
- Load repo-local skills for product-specific contracts.
- Treat prior plan status, comments, and summaries as claims to verify against live code.

## Phase 1: Authoring Workflow & Card Contract

When drafting or reviewing an implementation plan:

1. State current behavior and target behavior in observable terms.
2. Separate confirmed defects, working assumptions, and decisions requiring human input.
3. Trace affected data flows end-to-end: producer -> boundary -> persistence -> hydration -> consumer.
4. Order cards by dependency and split independently testable contracts.
5. Every card must specify:
   - Dependencies
   - Observable target contract
   - Implementation paths or symbols
   - Acceptance behavior
   - Behavior test that would fail against previous code
   - Safety boundaries
   - Status placeholder
6. Avoid vague acceptance phrases such as "implemented", "wired", or "works" without observable criteria.
7. Explicitly separate required work, deferred work, and out-of-scope boundaries.

## Phase 2: Execution & Verification Workflow

When executing or auditing an approved plan:

1. Map every card to its contract, implementation symbols, and behavior test before changing code.
2. Reclassify cards based on live code evidence, not historical markdown claims.
3. Trace cross-layer values through boundaries, persistence, hydration, state, and consumers.
4. Implement in strict dependency order; run focused unit/integration checks while iterating.
5. Classify failures before patching: runner, fixture, product invariant, or cleanup defect.
6. Run every required release gate in a supported environment (Windows native or WSL as specified).
7. Update status tables and documentation only from observed command outputs.

## Required Evidence Output

When authoring or executing a plan, include the canonical card table:

| Card | Contract | Implementation | Behavior Test | Result |
| --- | --- | --- | --- | --- |
| ID | Observable target | Paths or symbols | Test and assertion | Observed status |

When executing or validating a plan, include the gate validation table:

| Gate | Environment | Result | Evidence Or Failure |
| --- | --- | --- | --- |
| Command or check | Host and dependencies | Passed, failed, inconclusive, or not run | Output summary |

Always list remaining work, explicit deferrals, and unauthorized actions not taken. Never report an unrun gate as passed.

## Failure and Stop Policy

- Preserve the first causal failure and its diagnostics. An outer timeout is inconclusive unless an assertion failed first.
- Ask one focused question only when a missing choice materially changes the product contract, safety boundary, or authority scope.
- Stop immediately before production mutation, destructive work, or an unresolved contract conflict.
- Routine test failures and stale plan claims are normal work to diagnose, not reasons to interrupt the user.

## Completion

Finish only when every in-scope card has behavior evidence, every required gate has an honest status, owned test processes and data are cleaned up, and deferred items remain explicit.

## Evaluations (I/O Examples)

**Input:** "Create an implementation plan for a user setting persisted to database and reflected in two views"
**Expected:** Agent outputs dependency-ordered producer, schema/migration, persistence, hydration, and consumer cards with observable acceptance tests and verification matrix.

**Input:** "Execute this approved five-card plan"
**Expected:** Agent audits current code, implements in dependency order, runs tests, and produces both the card evidence table and the validation gate table.

**Input:** "Review this card: wire the value everywhere; acceptance: it works"
**Expected:** Agent rejects the vague card and rewrites it with named consumers, observable acceptance criteria, and a concrete behavior test.

## Do Not

- Do not hide uncertainty inside definitive acceptance language.
- Do not trust prior summaries or markdown checklists without verifying code.
- Do not combine unrelated contracts into untestable monolith cards.
- Do not patch product code to conceal invalid test fixtures or runner bugs.
- Do not mark failed, inconclusive, or unrun gates as passed.
- Do not authorize production migrations, deploys, or destructive actions without explicit user permission.

## References

- [plan-authoring-best-practices](references/plan-authoring-best-practices.md)
- [plan-execution-workflow](references/plan-execution-workflow.md)
- [plan-acceptance-evidence](references/plan-acceptance-evidence.md)
- [cross-layer-data-flow-verification](references/cross-layer-data-flow-verification.md)
- [playwright-e2e-fixture-contract-validation](references/playwright-e2e-fixture-contract-validation.md)
- [playwright-failure-classification](references/playwright-failure-classification.md)
- [completion-means-the-gate-ran](references/completion-means-the-gate-ran.md)

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
