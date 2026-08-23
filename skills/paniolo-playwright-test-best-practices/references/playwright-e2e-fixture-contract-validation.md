---
source-slug: playwright-e2e-fixture-contract-validation
source-hash: e2d6cf8186073e70da1dddaf6e61011ade6e3af93e3f864cc2ca0389d8a3271f
bundled: 2026-08-20
title: E2E Fixture Contract Validation
type: concept
tags:
- playwright
- testing
- fixtures
- debugging
- verification
updated: 2026-08-20
---

# E2E Fixture Contract Validation

Validate an end-to-end fixture against the product contract before changing product behavior. A
fixture that reaches the UI or API can still be invalid for the scenario it claims to represent.

## Contents

- [Fixture Contract](#fixture-contract)
- [Triage Workflow](#triage-workflow)
- [Evidence](#evidence)
- [Stop Conditions](#stop-conditions)
- [See Also](#see-also)

---

<a id="fixture-contract"></a>

## Fixture Contract

Check every dimension that the behavior depends on:

- identifiers have the required format and remain unique for the run;
- dimensions, counts, ranges, and enum values satisfy domain invariants;
- relationships point to the intended owner and existing parent records;
- the authenticated actor has the required role, grant, and row-level access;
- lifecycle state matches the scenario, including published, active, or pending status;
- dependent records exist in the correct order;
- disposable identifiers are recognized by local and global cleanup;
- protected shared fixtures cannot match a disposable cleanup rule.

Use a typed fixture builder or validated API when practical. Raw database inserts still need the
same validation and authorization assumptions documented explicitly.

---

<a id="triage-workflow"></a>

## Triage Workflow

1. Capture the first failed request or assertion, including response status and body.
1. Print or inspect the exact fixture values involved in the rejected contract.
1. Compare those values with boundary schemas, constraints, permissions, and domain rules.
1. Reproduce the boundary call with the smallest valid fixture.
1. Fix the builder, seed, session, or relationship when the fixture is wrong.
1. Change product validation only when the intended product contract is itself wrong.
1. Re-run the focused test and then the shared-state gate twice when repeatability is required.

---

<a id="evidence"></a>

## Evidence

Record the fixture source, actor, owned identifiers, relevant invariant values, boundary response,
cleanup path, and observed result. A fixture screenshot alone does not prove authorization or
persistence correctness.

For generated values, assert the important dimensions before the product action. This makes a
fixture defect fail near setup instead of appearing later as a misleading UI timeout.

---

<a id="stop-conditions"></a>

## Stop Conditions

Stop and request clarification when the plan, schema, and current product behavior disagree about
a domain invariant or authorization rule. Do not weaken constraints, skip row-level security,
grant broader permissions, add polling, or reuse unrelated shared data just to make the test pass.

---

<a id="see-also"></a>

## See Also

- Playwright Failure Classification
- Test-Owned Entities
- Plan Acceptance Evidence
