---
source-slug: plan-authoring-best-practices
source-hash: 2ec018de6b98d4bc67ae1f9d71343dc36369cd979ab56a042a1e18bd33a2b5f3
bundled: 2026-09-25
title: Plan Authoring Best Practices
type: concept
status: completed
tags:
- authoring
- plans
- testing
- verification
updated: 2026-08-20
---

# Plan Authoring Best Practices

A useful implementation plan is an executable contract, not a themed task list. It explains what
must become true, why the work is ordered, how each result will be observed, and what is outside
scope.

## Contents

- [Required Structure](#required-structure)
- [Card Contract](#card-contract)
- [Good And Bad Cards](#good-and-bad-cards)
- [Review Checklist](#review-checklist)
- [See Also](#see-also)

---

<a id="required-structure"></a>

## Required Structure

- State current and target behavior in observable terms.
- Separate confirmed defects from assumptions and open questions.
- Order cards by dependency: data and authorization, boundary validation, shared logic, consumers,
  then end-to-end coverage.
- Give each card concrete file or subsystem scope without treating paths as proof of completion.
- Write acceptance criteria as behavior that would fail against the old implementation.
- Name verification commands or gate categories, supported environments, and forbidden production
  actions.
- Separate deferred work from release-gate work.
- Include status and evidence areas for observed results after execution.

---

<a id="card-contract"></a>

## Card Contract

Each independently verifiable card should include:

```text
Card ID and title
Depends on
Current behavior
Target contract
Implementation scope
Acceptance behavior
Behavior test
Safety or rollback boundary
Status and evidence placeholder
```

Split a card when its acceptance test can pass for one contract while another contract remains
broken. Cross-layer cards must enumerate the producer, boundaries, state, and every consumer.

---

<a id="good-and-bad-cards"></a>

## Good And Bad Cards

Bad card:

```text
Wire saved labels into every display mode.
Acceptance: labels work everywhere.
```

This card has no current behavior, dependency, consumer inventory, or failing behavior test.

Better card:

```text
Card C3 — Hydrate saved labels in alternate display modes
Depends on: C1 persistence contract; C2 loading contract
Current behavior: the editor saves a label, but compact and presentation modes use a fallback.
Target contract: both modes read the persisted label from shared hydrated state.
Implementation scope: hydration mapper, shared selector, compact view, presentation view.
Acceptance: save a distinctive label, reload, and observe it in both modes.
Behavior test: one integration test asserts the reloaded state and each named consumer.
Safety: no production mutation; preserve fallback behavior for records without a label.
```

The better card can identify which layer or consumer remains disconnected.

---

<a id="review-checklist"></a>

## Review Checklist

Before handoff, confirm another agent can:

- find the relevant contracts and repository-specific skills;
- explain the dependency order;
- select a behavior test for every card;
- distinguish runner, fixture, product, and cleanup failures;
- report passed, failed, inconclusive, and not-run gates honestly;
- stop before unauthorized production or destructive changes.

---

<a id="see-also"></a>

## See Also

- Cross-Layer Data Flow Verification
- Plan Acceptance Evidence
- Completion Means the Gate Ran
- E2E Fixture Contract Validation
- Playwright Failure Classification
