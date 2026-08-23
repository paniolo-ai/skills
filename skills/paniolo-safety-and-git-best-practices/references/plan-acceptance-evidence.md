---
source-slug: plan-acceptance-evidence
source-hash: 3d4d706f40baf3553cd0b8157d30fa52638f384ef539b3c3b374690e3386c2a2
bundled: 2026-08-20
title: Plan Acceptance Evidence
type: concept
tags:
- authoring
- plans
- testing
- verification
updated: 2026-08-20
---

# Plan Acceptance Evidence

A plan card is complete only when its stated contract is implemented and its acceptance behavior
has been observed. A changed file, passing typecheck, status table, code comment, or prior agent
summary is not acceptance evidence by itself.

Use one evidence row per card while reviewing or implementing a plan:

```text
card -> contract -> implementation paths -> behavior test -> observed result
```

Trace every participating layer. A correct editor or API handler does not prove that saved data
reaches alternate screens, background consumers, exports, or presentation modes.

## Review workflow

1. Read the plan and every linked contract page.
1. Inspect the actual implementation; do not inherit the plan's status claims.
1. Identify schema, boundary validation, shared logic, UI consumers, scripts, and tests involved in
   each acceptance statement.
1. Add or locate a behavior test that would fail against the previous implementation.
1. Run the named gate and record its observed result.
1. Keep deferred work explicit and separate from completed cards.

Classify every verification result as **passed**, **failed**, **inconclusive**, or **not run**. A
killed outer process, infrastructure timeout, or reporter pipe error is inconclusive unless a test
assertion already established a failure.

For tests that mutate a shared environment, require two consecutive clean runs when repeatability
is part of the release contract. The second run must not depend on rows or processes left by the
first.

## Correction loop

When review finds a repeated agent mistake, repair the weakest reusable layer that could have
prevented it: a rule, skill, concept page, fixture helper, deterministic check, or test. Keep
product-specific guidance in that product's harness and put only reusable engineering guidance in
shared documentation.

## See also

- Cross-Layer Data Flow Verification
- Completion Means the Gate Ran
- E2E Fixture Contract Validation
- Playwright Failure Classification
- Test-Owned Entities
