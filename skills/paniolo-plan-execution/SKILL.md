---
name: paniolo-plan-execution
description: |
  Execute or audit an existing implementation plan by revalidating its claims against current code, tracking card-to-behavior evidence, diagnosing failures before patching, running required gates, and updating status honestly. Use when implementing a plan or reviewing another agent's plan execution. Do NOT use to author a new plan from scratch; load paniolo-plan-best-practices instead.
license: MIT
metadata:
  version: 0.1.1
tags:
- plans
- implementation
- verification
user-invocable: true
references:
- references/completion-means-the-gate-ran.md
- references/cross-layer-data-flow-verification.md
- references/plan-acceptance-evidence.md
- references/plan-execution-workflow.md
- references/playwright-e2e-fixture-contract-validation.md
- references/playwright-failure-classification.md
- references/playwright-test-owned-entities.md
---

**Requires:** file-read, repository search, file-write, and terminal access for verification gates.

**Companion:**
[paniolo-plan-best-practices](../paniolo-plan-best-practices/SKILL.md)

# Plan Execution

## Preconditions

- Read the plan, linked contracts, repository instructions, and repo-local skills.
- Inspect the dirty worktree and preserve unrelated changes.
- Treat prior status, comments, and summaries as claims to verify.

## Execution Workflow

1. Map every card to its contract, implementation symbols, and behavior test.
1. Reclassify cards from current evidence before changing code.
1. Trace cross-layer values through boundaries, persistence, hydration, state, and every consumer.
1. Implement in dependency order and run focused checks while iterating.
1. Diagnose end-to-end failures as runner, fixture, product, or cleanup defects before patching.
1. Run every required release gate in a supported environment.
1. Update plan status and documentation only from observed evidence.
1. Turn repeated mistakes into the weakest durable correction layer.

See [plan-execution-workflow](references/plan-execution-workflow.md).

## Required Evidence Output

Include this card table:

| Card | Contract | Implementation | Behavior Test | Result |
| --- | --- | --- | --- | --- |
| ID | Observable target | Paths or symbols | Test and assertion | Observed status |

Include a separate validation table:

| Gate | Environment | Result | Evidence Or Failure |
| --- | --- | --- | --- |
| Command or check | Host and dependencies | Passed, failed, inconclusive, or not run | Output summary |

Also list remaining work, explicit deferrals, cleanup evidence, and unauthorized actions not taken.
Never merge a focused pass with an unrun broad gate.

## Failure And Stop Policy

Preserve the first causal failure and its diagnostics. An outer timeout or reporter pipe error is
inconclusive unless an assertion failed first. Validate fixtures before weakening product
validation. Verify descendants and listeners before claiming runner cleanup.

Ask one focused question only when a missing choice changes the product contract or expands
authority. Stop before production mutation, destructive work, or a contract conflict that evidence
cannot resolve. Ordinary test failures and stale plan claims are work to diagnose, not reasons to
ask the user.

## Completion

Finish only when every in-scope card has behavior evidence, every required gate has an honest
status, owned processes and test data are cleaned up, documentation matches the landed state, and
deferred work remains explicit.

## Evaluations

**Input:** "Execute this approved five-card plan."

**Expected:** Audit current state, implement in dependency order, and return both required tables,
cleanup evidence, and remaining work with no unsupported completion claims.

**Input:** "Audit the previous agent's claim that all consumers were updated."

**Expected:** Inventory every consumer, map each to implementation and a behavior assertion, and
mark any missing or untested consumer incomplete.

**Input:** "The E2E API rejected the generated fixture, so relax the validator."

**Expected:** Load fixture-contract guidance, inspect the exact values and domain invariant, and
refuse to weaken product validation unless evidence shows the product contract is wrong.

## Do Not

- Do not author a new plan from scratch; load the companion authoring skill.
- Do not trust plan status, comments, or prior summaries as implementation evidence.
- Do not patch product code to conceal invalid fixtures or runner failures.
- Do not mark failed, inconclusive, or unrun gates as passed.

## References

- [plan-execution-workflow](references/plan-execution-workflow.md)
- [plan-acceptance-evidence](references/plan-acceptance-evidence.md)
- [cross-layer-data-flow-verification](references/cross-layer-data-flow-verification.md)
- [playwright-e2e-fixture-contract-validation](references/playwright-e2e-fixture-contract-validation.md)
- [playwright-failure-classification](references/playwright-failure-classification.md)
- [playwright-test-owned-entities](references/playwright-test-owned-entities.md)
- [completion-means-the-gate-ran](references/completion-means-the-gate-ran.md)
