---
name: paniolo-vitest-test-best-practices
description: |
  Vitest unit test authoring for this repo — setup, mocking, API handler testing, and common pitfalls for non-hook code. Use when the user asks to add, update, fix, or review unit tests for utilities, components, API handlers, or scripts. Do NOT use for React hook tests — load vitest-test-hook-best-practices instead.
license: MIT
metadata:
  version: 0.1.0
tags:
- vitest
- testing
references:
- 'wiki: sharp-shooter-wiki:vitest-colocate-make-fixtures → references/vitest-colocate-make-fixtures.md'
- 'wiki: sharp-shooter-wiki:vitest-extract-helpers-only-after-repetition → references/vitest-extract-helpers-only-after-repetition.md'
- 'wiki: sharp-shooter-wiki:vitest-forcecast-vs-make-fixtures → references/vitest-forcecast-vs-make-fixtures.md'
- 'wiki: sharp-shooter-wiki:vitest-mock → references/vitest-mock.md'
- 'wiki: sharp-shooter-wiki:vitest-one-canonical-make-helper → references/vitest-one-canonical-make-helper.md'
- 'wiki: sharp-shooter-wiki:vitest-test → references/vitest-test.md'
- 'wiki: sharp-shooter-wiki:vitest-testing-when-to-write → references/vitest-testing-when-to-write.md'
---

**Requires:** file-read, terminal (test runner). No network access needed.

**Related skills:**

- [`code-comment-best-practices/SKILL.md`](../paniolo-code-comment-best-practices/SKILL.md)
- [`typescript-lint-best-practices/SKILL.md`](../paniolo-typescript-lint-best-practices/SKILL.md)

## Full reference

[vitest-test](references/vitest-test.md) — load on demand
for
test structure, API handler execution, and general tradeoffs.

**Paniolo scan:** see
[[paniolo-wiki:scan-vitest-what-to-assert]]

[vitest-mock](references/vitest-mock.md) —
load on demand for mocking patterns, data-client stubs, API handler mock setup, or helper
architecture.

## When invoked

**Preconditions:**

- Read the source file and existing tests at the nearest behavior boundary.
- Apply [vitest-testing-when-to-write](references/vitest-testing-when-to-write.md) before creating a test file.
- Check rules for repo-wide constraints.
- Load `code-comment-best-practices` when you add named helpers or local guards.
- Load `vitest-test-hook-best-practices` for any hook test.

**Clarifying questions:**

- Default: first decide whether the behavior warrants a unit test. When it does, use the
  narrowest existing behavior-level test that can protect the contract.
- Always ask only when the file or behavior to cover is unclear.

**Error handling:**

- If `npm run test:unit` fails, report the error output verbatim, diagnose
  the root cause, and fix before declaring success.
- If the source file to test does not exist, stop and ask for the correct path.
- Do not skip or hide test failures — always report them.

## Execution workflow

1. Apply the two-part value gate: can a plausible defect make the test fail, and would that defect
   matter to a user, security boundary, or domain contract? If either answer is no, do not add the
   test.
2. Prefer an existing behavior- or feature-level test. Create a paired colocated same-basename test
   only when the source module owns a standalone behavioral contract.
3. Structure each test using the repo AAA conventions from the linked doc.
4. Use repo test patterns and mocking helpers before ad-hoc test doubles. Prefer shared `make*`
   helpers for typed fixtures; colocate feature-specific `make*` files with the feature folder (see
   [vitest-colocate-make-fixtures](references/vitest-colocate-make-fixtures.md));
   use `forceCast` only as an escape hatch (see
   [vitest-forcecast-vs-make-fixtures](references/vitest-forcecast-vs-make-fixtures.md)).
5. Keep assertions strict and deterministic.
6. Re-run targeted tests, then broaden as needed.

## First-pass checklist for new or heavily edited test files

- Keep explicit `// Arrange`, `// Act`, and `// Assert` comments in each `it`
  unless the linked docs explicitly allow an omission.
- Do not use `beforeEach`/`afterEach`/`beforeAll`/`afterAll` in `*.test.*`.
  Call `vi.resetAllMocks()` and any helper installers inside each `it`.
- Any named helper or local guard added in a test file or adjacent source file
  needs JSDoc with every `@param` and `@returns`, including tiny helpers such
  as `noop`, `makeProps`, `installMockBody`, `LocationProbe`, or
  `noopUnsubscribe`.
- Extract repeated numbers and strings into shared constants. This includes
  call counts like `1`, tab ids, pathnames, labels, payloads, and dataset keys.
- Prefer lowercase `describe` names for component/content tests when nearby
  files follow that convention.
- Use `mockTranslation()` for localized UI instead of ad-hoc
  translation-library mocks.
- Use `MemoryRouter` plus a probe component for navigation assertions instead
  of mocking `react-router-dom`.
- Default to bare `vi.mock("path")` + `vi.mocked(...)`. Avoid factory
  `vi.mock` unless the mock skill's exception rules apply.
- One canonical `make*` per fixture type — search `*.test-util.ts` before adding; import the
  existing helper; do not redefine the same builder in another spec (see
  [vitest-one-canonical-make-helper](references/vitest-one-canonical-make-helper.md)).
- Prefer shared `make*` helpers (`*.test-util.ts`, `Partial` overrides) for typed domain/DB rows;
  colocate feature-specific helpers with the feature folder; do not use `forceCast` when a `make*`
  helper exists.
- When a full prop, callback, mock, or store bundle repeats in one spec, extract a typed local
  `makeProps`/`makeHookProps`/`makeCallbacks` factory with `Partial` overrides. Keep it local
  until another spec needs the fixture, and create once per test so assertions use the same spies;
  see [vitest-extract-helpers-only-after-repetition](references/vitest-extract-helpers-only-after-repetition.md).
- Use `makeNoopFn()` for behaviorally irrelevant callbacks. Use `vi.fn<ActualSignature>()` when
  the test configures or verifies mock behavior; use a local structural adapter for overloaded
  callback types such as Zustand `Set`.
- For overloaded types such as Zustand `Set`, prefer `forceCast(...)` and captured call arrays
  over `vi.fn<...>()`, `Parameters<...>`, or unsafe raw `mock.calls` indexing.
- Use block-bodied event handlers in JSX mocks and bracket syntax for dataset
  access when needed to satisfy lint rules.
- After creating or changing tests, run `npm run lint` in addition to targeted
  test execution. Passing Vitest alone is not sufficient in this repo.

## Mocking strategy (details in docs)

Use non-factory `vi.mock("path")` + `vi.mocked(...)` by default. Escape hatches: `vi.spyOn` for
one-off overrides, `vi.doMock` for per-test runtime setup, factory patterns for advanced cases.

→ [vitest-mock](references/vitest-mock.md)

## Do not

- Do not violate repo-wide rules in rules.
- Do not add broad lint/type suppressions without explicit justification.
- Do not expand scope beyond requested task.
- Do not assert a mocked function's own implementation — assert observable behavior instead.
- Do not mock `effect` at module level. Mock your own boundary and return real `Effect` values.
- Do not treat `vi.hoisted()` or `vi.importActual()` as baseline patterns — they are exceptions.
- Do not use `beforeEach`/`afterEach`/`beforeAll`/`afterAll` in `*.test.*` (`jest/no-hooks`).
- Do not hand-roll translation-library module mocks when a shared `mockTranslation`
  test util fits the scenario.
- Do not mock `react-router-dom` when a `MemoryRouter` render exercises the
  behavior directly.
- Do not stop after a targeted Vitest run when a new or edited test file still
  needs `npm run lint` validation.
- Do not add colocated unit tests for constants-only modules (`*-constants.ts` and similar files
  that export literals only). Test behavior in the functions that use those constants.
- Do not add runtime tests for type-only modules or tests that only assert a dynamic import
  succeeds.
- Do not add dedicated tests for fixtures, spies, fake stores, or other test utilities. Their
  production-behavior consumers provide the useful contract.
- Do not create a test file merely because source refactoring extracted a function into its own
  module. Colocation determines where a justified test lives; it does not require a test.

## Lint and data normalization

See [vitest-test](references/vitest-test.md) for
the full lint checklist, `it.each`, and normalization guidance.

## Validation commands

```bash
npm run test:unit -- path/to/file.test.ts   # targeted (fastest)
npm run test:unit                            # all tests (before PR)
npm run lint                                 # required for new/edited test files
```

## Success criteria

- All targeted tests pass.
- `npm run lint` passes for new or edited test files, or any skip is explained.
- Changes follow this skill's conventions and project rules.
- Test commands are run and results reported (or skip is explicitly justified).
- Results clearly summarize behavior impact and remaining risks.

## References

- Full test reference: [vitest-test](references/vitest-test.md)
- Repo-wide rules: rules