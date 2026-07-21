---
source-wiki: sharp-shooter-wiki
source-slug: vitest-test
source-hash: 952ad641428f146b5c16d8f94b50f0c6ff611f207e69adba9258f7c727ee058e
bundled: 2026-07-20
title: Authoring — Vitest unit testing
type: index
tags:
- index
- authoring
- vitest-test
updated: 2026-06-18
---

# Vitest unit testing (authoring)

Operational reference for vitest unit testing — loaded from skills and agents.
Repo hub: `docs/testing/vitest-test-best-practices.md` (thin router during wiki migration).

## Pages

- AAA placement conventions
- AAA Testing Pattern (Arrange, Act, Assert)
- `act` vs `waitFor`
- API Handler Testing
- ❌ `Array#sort()` Instead of `Array#toSorted()`
- ❌ `as any` in Test Bodies
- Assert Error Semantics, Not Just \"Throws\"
- ❌ Assuming Utility Output Without Checking the Implementation
- ❌ Async Race Conditions
- Behavior vs Implementation Assertions
- Where to place `make*` helpers
- Core Setup
- Data shape & normalization guidance
- Deterministic Async
- Edge cases
- ❌ Duplicated Literal Test Data
- Extract Helpers Only After Repetition
- Example: mocking a fetch
- First-pass lint checklist for new test files
- How to run the helper script
- forceCast vs make\\\* Test Fixtures
- ⚠️ Lint Disable Comments
- ❌ Magic Numbers and Strings
- ❌ Manual Global Overrides
- Minimal `ReadonlyContext` for small helpers
- Vitest mocking
- `MockRow<T>` and `exactOptionalPropertyTypes`
- One canonical `make*` per fixture type
- Practical rules
- Quick checklist
- Prefer shared `make*` helpers
- References
- Quick-Start Routing Guide
- `ERR_REQUIRE_ESM` / `html-encoding-sniffer` / `@exodus/bytes` during Vitest startup
- Running the Handler and Asserting Results
- Script / pure-logic module testing
- Test Helpers: Use Existing Factories
- ❌ `toEqual` instead of `toStrictEqual`
- 🛠️ Type-Cast Helpers
- Use `forceCast` only as an escape hatch
- Validation Commands
- When to Write a Test (and When NOT to)

## See also

- Authoring index
- Repo hub: `docs/testing/vitest-test-best-practices.md`