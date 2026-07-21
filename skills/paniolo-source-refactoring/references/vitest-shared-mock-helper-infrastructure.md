---
source-wiki: sharp-shooter-wiki
source-slug: vitest-shared-mock-helper-infrastructure
source-hash: aa49debe121c541b9add84eaf41036e88fc8810bcb9faa0b836eb8ab5a98467a
bundled: 2026-07-20
title: Shared Mock Helper Infrastructure
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
// react/src/event/manage/test-utils/mockUseSlideManagerView.ts
import { vi } from "vitest";

let mockFn: ReturnType<typeof vi.fn> | undefined = undefined;

/**
 * Set up the mock for useSlideManagerView.
 * Must be called explicitly in each test before using the hook.
 * @returns The mock function for inspection
 */
export default function mockUseSlideManagerView(): ReturnType<typeof vi.fn> {
	vi.resetModules();
	mockFn = vi.fn();
	vi.doMock("@/react/event/manage/slide/useSlideManagerView", () => ({ default: mockFn }));
	return mockFn;
}

export function getMockFn(): ReturnType<typeof vi.fn> | undefined {
	return mockFn;
}
```

**Global mock storage with `vi.hoisted()`** — use `vi.hoisted(() => ({ mockFn: undefined }))` to
share mock state across helper files. See real examples in `react/src/form/test-util.ts`.

**Helper module rules:**

1. No top-level `vi.mock` calls in helper modules. Export a callable function (`mockFoo()`) that
   calls `vi.doMock` when invoked.
2. No lint disable comments in test files — test-util files may use disables only when unavoidable.
3. See real examples in `react/src/form/test-util.ts` and
   `react/src/lib/supabase/client/getSupabaseClient.test-util.ts`.

## See also

- Vitest mocking (authoring) index