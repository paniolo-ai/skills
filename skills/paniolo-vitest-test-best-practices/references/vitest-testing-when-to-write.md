---
source-wiki: sharp-shooter-wiki
source-slug: vitest-testing-when-to-write
source-hash: 786b4a9ec399d6bb13cd96021f4531eb558543d523c865cefd6a3903f608ef4b
bundled: 2026-08-01
title: When to Write a Test (and When NOT to)
type: concept
tags:
- authoring
- vitest
- testing
updated: 2026-07-12
---

# When to Write a Test (and When NOT to)

Write a unit test only when it protects behavior whose failure matters. File
colocation tells you where a justified test belongs; it does not require one
test file for every source module.

## Value gate

Answer both questions before adding a unit test:

1. Can a plausible defect make this test fail?
2. Would that defect matter to a user, security boundary, data contract, or
   nontrivial domain behavior?

If either answer is no, do not add the test by default. AAA structure,
assertion count, test count, and line coverage do not establish value.

Prefer an existing feature- or behavior-level test when it can protect the
contract. Create a paired same-basename test only when the source module owns a
standalone behavioral contract.

**When NOT to write a test:**

- **Trivial getters / pure config objects** — a function that returns a constant or reads a single
  field provides no falsifiable behavior worth asserting.
- **Constants-only modules** — do not add colocated `*.test.ts` for files that only export named
  constants (for example `*-constants.ts`, `shared-constants.ts`). There is no behavior to
  exercise; consumers and integration tests already depend on those values. Import the constants
  in real specs instead of asserting `MAX_ATTEMPTS === 2`.
- **Generated or vendored code** — files under `generated/` or third-party wrappers you don't own.
- **Test-util helpers themselves** — helpers like `asApiResponse` or `makeCtx` are
  infrastructure, not behavior; they don't need their own specs.
- **Type-only modules** — runtime imports cannot verify TypeScript contracts. Use the compiler or
  a dedicated compile-time assertion when a type contract needs explicit verification.
- **One-liner re-exports** — if the only logic is `export default someLib.method`, the test would
  just verify that JS imports work.
- **Mechanical extraction** — moving a private helper into its own file does not create a new
  testing obligation. Keep coverage at the behavior boundary that consumes it.

If you find yourself writing a test that asserts an import exists, that a constant equals itself,
or that `LAST_ATTEMPT_INDEX` is `MAX_ATTEMPTS - 1`, skip it.

If a function or its result is mocked in a test, avoid adding a separate unit test that asserts
the mocked function's own implementation; instead, assert the behavior of the system under test
that depends on that mock.

## See also

- Vitest unit testing (authoring) index
