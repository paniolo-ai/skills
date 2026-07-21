---
source-wiki: sharp-shooter-wiki
source-slug: vitest-domock
source-hash: 485f127a7e4c690e8a32520a58c40b338c94d7912c6892d2a4c3d18feb3a0f18
bundled: 2026-07-20
title: '`vi.doMock()` - Runtime Exception Path'
type: concept
tags:
- authoring
- vitest
- testing
- mocking
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# `vi.doMock()` - Runtime Exception Path

Use `vi.doMock()` only when non-factory top-level `vi.mock("path")` cannot express the test setup.
Typical case: per-test runtime-dependent mocking before importing the SUT.

Preferred flow for `vi.doMock()`:

1. Create a local `async init()` helper inside `describe`.
2. Call `vi.resetModules()` inside `init()` before imports.
3. Install `vi.doMock(...)` in `init()`.
4. Dynamically `import()` the SUT and mocked dependency in `init()`.
5. Return all handles needed by the test.

Do not use `vi.doMock()` as a default replacement for top-level non-factory `vi.mock`.

## See also

- Vitest mocking (authoring) index