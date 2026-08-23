---
name: paniolo-plan-best-practices
description: |
  Author or review executable implementation plans with dependency-ordered cards, observable acceptance criteria, verification gates, explicit deferrals, and safety boundaries. Use when creating or improving a plan before implementation begins. Do NOT use to execute an existing approved plan; load paniolo-plan-execution instead.
license: MIT
metadata:
  version: 0.1.1
tags:
- plans
- authoring
user-invocable: true
references:
- references/completion-means-the-gate-ran.md
- references/cross-layer-data-flow-verification.md
- references/plan-acceptance-evidence.md
- references/plan-authoring-best-practices.md
- references/playwright-e2e-fixture-contract-validation.md
---

**Requires:** file-read and repository search. Terminal access is useful for verifying commands.

**Companion:**
[paniolo-plan-execution](../paniolo-plan-execution/SKILL.md)

# Plan Best Practices

## Preconditions

- Read repository instructions, existing plans, linked contracts, and the dirty worktree.
- Search for current implementation and tests before describing a defect as confirmed.
- Load repo-local skills for product-specific contracts.

## Authoring Workflow

1. State current and target behavior in observable terms.
1. Separate confirmed defects, assumptions, and decisions that require user input.
1. Trace affected data from producer through every boundary and consumer.
1. Order cards by dependency and split independently testable contracts.
1. Give every card a behavior test that would fail against the old implementation.
1. Define supported environments, release gates, safety boundaries, and cleanup evidence.
1. Separate required, deferred, and out-of-scope work.
1. Add a status and evidence table for later execution.

See [plan-authoring-best-practices](references/plan-authoring-best-practices.md) and
[cross-layer-data-flow-verification](references/cross-layer-data-flow-verification.md).

## Card Contract

Every card must name its dependencies, current behavior, target contract, implementation scope,
acceptance behavior, behavior test, safety boundary, and status placeholder. Avoid acceptance words
such as "implemented," "wired," or "works" without an observable outcome.

See [plan-acceptance-evidence](references/plan-acceptance-evidence.md).

## Output Contract

Create the plan in the repository's canonical `plan-*` location and format. Include:

- objective, confirmed gaps, dependency order, and implementation cards;
- a verification matrix with named gate categories and supported environments;
- status and evidence placeholders;
- deferred work, unresolved decisions, and forbidden production actions.

Summarize dependencies and unresolved decisions. Do not begin implementation unless the user also
requested execution.

## Error And Clarification Policy

Ask one focused question only when a missing choice materially changes the product contract or
safety boundary. Otherwise state the assumption and continue. Stop if the requested plan would
authorize destructive or production work beyond the user's scope. If repository evidence
contradicts the request, show the evidence and preserve the conflict as an explicit decision.

## Evaluations

**Input:** "Create a plan for a value that must persist and appear in three display modes."

**Expected:** Dependency-ordered producer, boundary, persistence, hydration, consumer, and behavior
test cards; each display mode appears in the evidence matrix.

**Input:** "Review this card: wire the value everywhere; acceptance: it works."

**Expected:** Reject the vague card and rewrite it with dependencies, named consumers, observable
acceptance behavior, and a test that fails against the old behavior.

**Input:** "Execute the approved plan and audit the previous agent's completion claims."

**Expected:** Hand off to
[paniolo-plan-execution](../paniolo-plan-execution/SKILL.md); do not implement under this skill.

## Do Not

- Do not execute an existing approved plan; load the companion execution skill.
- Do not hide uncertainty inside definitive acceptance language.
- Do not combine unrelated contracts into one untestable card.
- Do not authorize production migrations, deploys, or destructive operations through plan prose.

## References

- [plan-authoring-best-practices](references/plan-authoring-best-practices.md)
- [plan-acceptance-evidence](references/plan-acceptance-evidence.md)
- [cross-layer-data-flow-verification](references/cross-layer-data-flow-verification.md)
- [playwright-e2e-fixture-contract-validation](references/playwright-e2e-fixture-contract-validation.md)
- [completion-means-the-gate-ran](references/completion-means-the-gate-ran.md)
