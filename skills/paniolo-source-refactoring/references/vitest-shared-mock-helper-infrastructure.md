---
source-wiki: sharp-shooter-wiki
source-slug: vitest-shared-mock-helper-infrastructure
source-hash: b03557b803ba87741e4333c0b6546ff89e8c40406cd5de1e69ab11f8f0bbc2db
bundled: 2026-07-24
title: Shared Mock Helper Infrastructure
type: concept
tags:
- authoring
- vitest
- testing
- mocking
updated: 2026-06-18
---

# Shared Mock Helper Infrastructure

Three patterns exist; pick the right one upfront:

| Situation                                            | Pattern                                     | Location         |
| ---------------------------------------------------- | ------------------------------------------- | ---------------- |
| Single test file needs fresh module imports per test | `async init()` inside `describe`            | Inline           |
| Multiple test files mock the same module             | Callable `mockFoo()` function               | `*.test-util.ts` |
| Multiple helper files share the same mock state      | `vi.hoisted()` state + `mockFoo()` + getter | `*.test-util.ts` |

Treat `vi.hoisted()` as a code smell by default in this repo. Most tests should use top-level
`vi.mock("path")` plus per-test `vi.mocked(...)` setup. Reach for `vi.hoisted()` only when hoist
timing is required for shared mock state across helper modules.

Start with `async init()`. Extract to a callable helper only when two or more test files need the
same mock. Add `vi.hoisted()` only when a second helper file needs to read or configure the same
mock function.

**Callable mock setup functions:**

```typescript
// react/src/item/manage/test-utils/mockUseItemManagerView.ts
import { vi } from "vitest";

let mockFn: ReturnType<typeof vi.fn> | undefined = undefined;

/**
 * Set up the mock for useItemManagerView.
 * Must be called explicitly in each test before using the hook.
 * @returns The mock function for inspection
 */
export default function mockUseItemManagerView(): ReturnType<typeof vi.fn> {
	vi.resetModules();
	mockFn = vi.fn();
	vi.doMock("@react/item/manage/useItemManagerView", () => ({ default: mockFn }));
	return mockFn;
}

export function getMockFn(): ReturnType<typeof vi.fn> | undefined {
	return mockFn;
}
```

**Global mock storage with `vi.hoisted()`** — use `vi.hoisted(() => ({ mockFn: undefined }))` to
share mock state across helper files. See the helper modules in your own `test-util.ts` files.

**Helper module rules:**

1. No top-level `vi.mock` calls in helper modules. Export a callable function (`mockFoo()`) that
   calls `vi.doMock` when invoked.
2. No lint disable comments in test files — test-util files may use disables only when unavoidable.
3. Keep such examples beside the feature that owns them (for example a form `test-util.ts`
   and a `getDbClient.test-util.ts` in the data-access layer).

## See also

- Vitest mocking (authoring) index