---
source-wiki: sharp-shooter-wiki
source-slug: vitest-forceCast-and-the-installStore-selector-dispatch-pattern
source-hash: 934f0d2c39ef71c82589c35568c55aa29fc070ce90d992ac4e1cc1800834693f
bundled: 2026-07-20
title: '`forceCast` and the `installStore` Selector Dispatch Pattern'
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

# `forceCast` and the `installStore` Selector Dispatch Pattern

When mocking `useAppStore` with `vi.mocked(...).mockImplementation(...)`, use `forceCast` to
invoke the real selector against typed mock state. Avoid `String(selector).includes(...)` string
inspection — it breaks with minification and function renames.

```tsx
import forceCast from "@/react/lib/test-utils/forceCast";

const mockFetchUserLibrary = vi.fn(() => Effect.sync(() => undefined));
const mockState = { fetchUserLibrary: mockFetchUserLibrary };

function installStore() {
	mockedUseAppStore.mockImplementation((selector: unknown) =>
		forceCast<(state: typeof mockState) => unknown>(selector)(mockState),
	);
	return { fetchUserLibrary: mockFetchUserLibrary };
}
```

For **typed domain rows** (`EventParticipant`, `EventPublic`, Supabase rows), prefer shared
`make*` helpers from colocated `*.test-util.ts` files with `Partial` overrides — see
vitest-forcecast-vs-make-fixtures.

Use `forceCast<T>(value)` only when you need to coerce past a type mismatch without a `make*`
helper — partial store states, synthetic event stubs, invalid error-path payloads, selector
dispatch, and similar escape hatches. Never use inline `as unknown as T`.

## See also

- Vitest mocking (authoring) index