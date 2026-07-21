---
name: paniolo-vitest-test-hook-best-practices
description: |
  Complete guide for testing React hooks — renderHook, Documentation by Harness, installStore, fixtures, subscription patterns, lint/compiler traps, and pre-completion checklist. Read [[sharp-shooter-wiki:vitest-hook]] for the full reference.
license: MIT
metadata:
  version: 0.1.0
tags:
- vitest
- testing
- hooks
references:
- 'wiki: sharp-shooter-wiki:vitest-colocate-make-fixtures → references/vitest-colocate-make-fixtures.md'
- 'wiki: sharp-shooter-wiki:vitest-documentation-by-harness → references/vitest-documentation-by-harness.md'
- 'wiki: sharp-shooter-wiki:vitest-extract-helpers-only-after-repetition → references/vitest-extract-helpers-only-after-repetition.md'
- 'wiki: sharp-shooter-wiki:vitest-forcecast-vs-make-fixtures → references/vitest-forcecast-vs-make-fixtures.md'
- 'wiki: sharp-shooter-wiki:vitest-hook → references/vitest-hook.md'
- 'wiki: sharp-shooter-wiki:vitest-mock → references/vitest-mock.md'
- 'wiki: sharp-shooter-wiki:vitest-test → references/vitest-test.md'
---

**Requires:** file-read, terminal (test runner). No network access needed.

**Also load:**
[`vitest-test-best-practices/SKILL.md`](../paniolo-vitest-test-best-practices/SKILL.md) —
hook tests must follow all conventions from the general skill (AAA pattern, named constants,
mocking order, `toStrictEqual`, lint rules) in addition to the hook-specific rules here.

# Unit Testing — React Hooks

## Use When

Use this skill when:

- Writing or editing tests for any `use*.ts` / `use*.tsx` hook.
- Validating hook behavior, side effects, and cleanup logic.

Execution workflow:

1. MANDATORY: Every hook test file MUST include both `renderHook` tests and at least one Harness
   component. A Harness is always required — even when `renderHook` covers all behavior (see
   "Documentation by Harness" in the full reference). This dual approach ensures behavioral
   correctness and clear documentation of how the hook is used in real UI.
2. Keep tests colocated and use shared wrappers/utilities.
3. Verify async behavior and cleanup paths explicitly.
4. Run targeted hook tests first, then broader checks.

Output requirements:

- State which behaviors were covered by `renderHook` vs Harness tests.
- Report exact test/lint commands run.

## Key Requirements

- Use **separate `describe` blocks** for renderHook tests and Harness tests:
  `describe("useMyHook — Harness", ...)` and `describe("useMyHook — renderHook", ...)`.
- Follow hook testing conventions in project docs.
- Harness tests MUST assert all properties returned by the hook (state values, derived values,
  and handler functions). If a return value is not directly observable, expose it via the
  Harness UI (for example render the value into the DOM or wire handlers to buttons) so
  the test can verify the behavior. This ensures the Harness documents and validates the
  hook contract end-to-end.

  See the detailed documentation and examples in the docs:
  [vitest-documentation-by-harness](references/vitest-documentation-by-harness.md).

- Prefer deterministic assertions over timing-based checks.
- Re-use existing types for all test data objects and fixtures. Prefer shared `make*` helpers with
  `Partial<T>` overrides for complete rows; colocate feature-specific `make*` with the feature
  folder; use `Writeable<T>` / `Partial<T>` for partial typed props; reserve `forceCast` for
  intentional bypasses (see
  [vitest-forcecast-vs-make-fixtures](references/vitest-forcecast-vs-make-fixtures.md) and
  [vitest-colocate-make-fixtures](references/vitest-colocate-make-fixtures.md)).
- When a hook's full callback or option bundle repeats within one spec, use a local typed
  `makeHookProps` or `makeCallbacks` factory with `Partial` overrides. Create it once per test so
  the asserted spies are the same instances passed to the hook; see
  [vitest-extract-helpers-only-after-repetition](references/vitest-extract-helpers-only-after-repetition.md).
- **Always use the AAA pattern** — every `it` block must have explicit `// Arrange`, `// Act`,
  `// Assert` comments. For tests that only verify initial state, use `// Arrange + Act` on the
  setup line and `// Assert — no Act: verifying initial render state only`. For multi-step tests,
  label each step `// Act — cycle 1`, `// Act — cycle 2`, etc.
- **`vi.resetAllMocks()` in shared setup helpers** — any `setup*()` helper shared across tests
  must call `vi.resetAllMocks()` as its first line to prevent mock call-count leakage.
- Prefer non-factory `vi.mock("path")` + `vi.mocked(...)` and static top-level imports for test
  doubles when possible; avoid dynamic `import()` in tests unless the behavior specifically requires
  runtime module resolution.
- **Mocked sub-hook setters** — when the hook under test calls a mocked sub-hook (e.g.
  `useItemTags`), its returned setters are `vi.fn()` stubs that don't update React state. Assert
  on a module-level named spy instead of expecting DOM changes.
- **`no-negated-condition` in JSX** — write `x === undefined ? "absent" : "present"`, not
  `x !== undefined ? "present" : "absent"`.

## References

- Full hook testing guide:
  [vitest-hook](references/vitest-hook.md)
- General testing guide:
  [vitest-test](references/vitest-test.md)
- Mocking guide:
  [vitest-mock](references/vitest-mock.md)
- Non-hook tests:
  [../vitest-test-best-practices/SKILL.md](../paniolo-vitest-test-best-practices/SKILL.md)

## Validation

```bash
npm run test:unit -- path/to/useMyHook.test.tsx
npx oxlint --config .oxlintrc.json --type-aware path/to/useMyHook.test.tsx
npm run test:unit
```

## Do Not

- Do not violate repo-wide rules in rules.
- Do not add broad lint/type suppressions without explicit justification.
- Do not expand scope beyond the requested task without calling it out.

## Success Criteria

- Changes follow this skill's conventions and project rules.
- Relevant validation commands are run, or skipped with a clear reason.
- Results clearly summarize behavior impact and remaining risks.

## Skill Handoffs

- The general skill is a hard dependency — it must always be loaded alongside this one (see **Also
  load** at the top).
- If hook behavior depends on routing or page components, also load `react-best-practices`.