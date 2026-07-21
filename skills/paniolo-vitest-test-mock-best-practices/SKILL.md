---
name: paniolo-vitest-test-mock-best-practices
description: |
  Vitest mocking guidance for this repo — module mock strategy, Supabase and Postgrest response stubs, API-handler mock setup, shared mock helpers, *forceCast* store selectors, and common mock pitfalls. Use when the user asks how to mock something in Vitest, wants mock-heavy unit tests added or fixed, or needs a review of mock strategy. Do NOT use for general non-mock test structure, AAA guidance, or hook Harness requirements.
license: MIT
metadata:
  version: 0.1.0
tags:
- vitest
- testing
- mocking
references:
- 'wiki: sharp-shooter-wiki:vitest-api-handler-mock-setup → references/vitest-api-handler-mock-setup.md'
- 'wiki: sharp-shooter-wiki:vitest-colocate-make-fixtures → references/vitest-colocate-make-fixtures.md'
- 'wiki: sharp-shooter-wiki:vitest-domock → references/vitest-domock.md'
- 'wiki: sharp-shooter-wiki:vitest-forceCast-and-the-installStore-selector-dispatch-pattern → references/vitest-forceCast-and-the-installStore-selector-dispatch-pattern.md'
- 'wiki: sharp-shooter-wiki:vitest-mock → references/vitest-mock.md'
- 'wiki: sharp-shooter-wiki:vitest-mock-vs-spyon → references/vitest-mock-vs-spyon.md'
- 'wiki: sharp-shooter-wiki:vitest-mocking-nodejs-built-ins → references/vitest-mocking-nodejs-built-ins.md'
- 'wiki: sharp-shooter-wiki:vitest-non-factory-mock → references/vitest-non-factory-mock.md'
- 'wiki: sharp-shooter-wiki:vitest-react-dom-mocking-flushsync → references/vitest-react-dom-mocking-flushsync.md'
- 'wiki: sharp-shooter-wiki:vitest-shared-mock-helper-infrastructure → references/vitest-shared-mock-helper-infrastructure.md'
- 'wiki: sharp-shooter-wiki:vitest-supabase-postgrest-mocking → references/vitest-supabase-postgrest-mocking.md'
- 'wiki: sharp-shooter-wiki:vitest-test → references/vitest-test.md'
---

**Requires:** file-read, terminal (test runner). No network access needed.

**Related skills:**
[`vitest-test-best-practices/SKILL.md`](../paniolo-vitest-test-best-practices/SKILL.md),
[`vitest-test-hook-best-practices/SKILL.md`](../paniolo-vitest-test-hook-best-practices/SKILL.md)

# Vitest Test Mock Best Practices

## Use When

- Writing or updating tests whose main challenge is mocking dependencies.
- Choosing between `vi.mock`, `vi.spyOn`, `vi.doMock`, or helper-based stubs.
- Reviewing whether a test's mock seam is appropriate.

## Full reference

[vitest-mock](references/vitest-mock.md)
— load on demand for the canonical mock patterns and deep-link targets below.

[vitest-test](references/vitest-test.md)
— load for AAA guidance, async assertions, validation commands, and broader
test structure.

## When invoked

**Preconditions:**

- Read the existing test file and the SUT imports first.
- Check rules for repo-wide constraints.
- Load `vitest-test-hook-best-practices` for hook tests.

**Clarifying questions:**

- Default: use non-factory `vi.mock("path")`, per-test `vi.mocked(...)`, and one seam at a time.
- Always ask only when the file or dependency to mock is unclear.

**Error handling:**

- If a requested file does not exist, stop and report the missing path.
- If the chosen mock pattern requires runtime imports or factories, explain why
  the default non-factory pattern is insufficient.
- Do not hide test failures caused by mock changes; report and fix them.

## Execution workflow

1. Verify the SUT's real imports before adding any mocks.
2. Choose the narrowest viable mock seam using the order below.
3. Reuse repo helpers like `asPostgrestResponse(...)`, `mockCreateSupabaseClient(...)`,
   `makeCtx(...)`, or existing `*.test-util.ts` files before inventing new stubs.
4. Keep mock setup readable at the test call site unless sharing is clearly
   justified across multiple tests or files.
5. Run the narrowest relevant validation first, then broaden only if needed.

## Key patterns

### Mocking order

1. Default to [vitest-non-factory-mock](references/vitest-non-factory-mock.md).
2. Use [vitest-mock-vs-spyon](references/vitest-mock-vs-spyon.md)
   only for one-off partial overrides.
3. Use [vitest-domock](references/vitest-domock.md)
   only when runtime-dependent mocking must happen before importing the SUT.
4. Use factories, `vi.importActual`, or `vi.hoisted` only for explicit advanced
   cases that the default pattern cannot express.

### Route by scenario

- **React Router components** — Use `MemoryRouter` wrapper instead of mocking; see
  [vitest-mock](references/vitest-mock.md)
- **Localized UI** — Use `mockTranslation()` before writing custom `react-i18next` mocks.
- **Default-export app components (function declarations)** — Bare `vi.mock("path")`, import the
  default, then `vi.mocked(Component).mockImplementation(...)` per test; see
  [vitest-mock](references/vitest-mock.md)
- **Lazy shell/content wrappers** — Mock the real lazily imported body component,
  not the registry map, when the direct child is the seam under test.
- **Supabase query mocks** — [vitest-supabase-postgrest-mocking](references/vitest-supabase-postgrest-mocking.md)
- **Shared helper modules** — [vitest-shared-mock-helper-infrastructure](references/vitest-shared-mock-helper-infrastructure.md)
- **Store selector stubs** —
  [vitest-forceCast-and-the-installStore-selector-dispatch-pattern](references/vitest-forceCast-and-the-installStore-selector-dispatch-pattern.md)
- **API handler mocks** — [vitest-api-handler-mock-setup](references/vitest-api-handler-mock-setup.md)
- **Node built-ins** — [vitest-mocking-nodejs-built-ins](references/vitest-mocking-nodejs-built-ins.md)
- **react-dom / flushSync** — [vitest-react-dom-mocking-flushsync](references/vitest-react-dom-mocking-flushsync.md)

### Repo-specific defaults

- Mock only modules the SUT directly imports.
- Prefer one mock seam per test.
- Do not mock `effect` at module level; mock your own boundary module instead.
- Prefer shared `make*` helpers for typed fixtures before `forceCast`; colocate feature-specific
  `make*` with the feature folder (see
  [vitest-colocate-make-fixtures](references/vitest-colocate-make-fixtures.md)).
- Avoid inline `as any`; use `vi.mocked(...)`, `forceCast(...)`, and shared
  test helpers.
- Keep `vi.hoisted()` rare and justified.
- For router assertions, prefer `MemoryRouter` plus a probe component over
  mocking `react-router-dom` exports.
- For overloaded store setter or subscription types, prefer `forceCast(...)`
  and captured call arrays over `vi.fn<...>()` or raw `mock.calls` indexing.
- Use shared constants for repeated ids, payloads, pathnames, labels, and call
  counts so mocks and assertions stay aligned and lint-clean.

## Validation

```bash
npm run test:unit -- path/to/file.test.ts
npm run lint
```

## Do Not

- Do not hand-craft Postgrest response objects inline when
  `asPostgrestResponse(...)` fits.
- Do not use factory `vi.mock` by default.
- Do not use `vi.mock<typeof import("react-i18next")>(...)` factory mocks by
  default when `mockTranslation()` fits.
- Do not use `beforeEach`/`afterEach`/`beforeAll`/`afterAll` in `*.test.*` (`jest/no-hooks`);
  configure `vi.mocked(...)` inside each `it` or call a small helper from each test.
- Do not add mocks for modules the SUT does not import.
- Do not introduce shared helper modules when a local setup block is clearer.
- Do not duplicate general AAA or hook Harness guidance here; defer to the
  companion skills and docs.

## Success Criteria

- The chosen mock seam matches the SUT's actual dependency boundary.
- The change follows the repo's preferred mock order and helper patterns.
- Validation was run and reported, or skipped with a clear reason.
- The result leaves a future reader with a clearer, not more magical, test.

## References

- Full mock reference: [vitest-mock](references/vitest-mock.md)
- Full test reference: [vitest-test](references/vitest-test.md)
- Repo-wide rules: rules
- [vitest-test-best-practices/SKILL.md](../paniolo-vitest-test-best-practices/SKILL.md)
- [vitest-test-hook-best-practices/SKILL.md](../paniolo-vitest-test-hook-best-practices/SKILL.md)