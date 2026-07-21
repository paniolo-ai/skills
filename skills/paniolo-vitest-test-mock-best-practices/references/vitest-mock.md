---
source-wiki: sharp-shooter-wiki
source-slug: vitest-mock
source-hash: 02eb41488ee259aaadbccb4392714f81d77dd0b22d70f9789529946927b3920b
bundled: 2026-07-20
title: Authoring — Vitest mocking
type: index
tags:
- index
- authoring
- vitest-mock
updated: 2026-06-18
---

# Vitest mocking (authoring)

Operational reference for vitest mocking — loaded from skills and agents.
Repo hub: `docs/testing/vitest-test-mock-best-practices.md` (thin router during wiki migration).

## Pages

- Setup: Mock External Modules
- ❌ `as any` Casts on Mocked Functions
- At A Glance
- Avoid Over-Mocking Pure Logic
- Best Practices For Test Doubles
- Choose One Mocking Seam
- Choosing The Right Double
- ❌ Collecting Mock Call Arguments in a Side-Effect Array
- `vi.doMock()` - Runtime Exception Path
- ESM & Effect Mocking
- `forceCast` and the `installStore` Selector Dispatch Pattern
- When to Use the `vi.mock` Factory Pattern (Required Guidance)
- `vi.mock()` vs `vi.spyOn()` - Default to `vi.mock()`
- Mocking Effect Functions
- ❌ Mocking Modules the Code Doesn't Import
- ❌ Mocking Node.js Built-ins (`node:fs/promises`, `node:path`, etc.)
- Mocking Order
- Module Cache Isolation
- Never Mock an Entire Shared Library
- Non-Factory `vi.mock` Pattern (Required)
- `react-dom` mocking (`flushSync`)
- References
- Repo-specific defaults that prevent first-pass failures
- Quick-Start Routing Guide
- Shared Mock Helper Infrastructure
- Stubs Vs Mocks
- Supabase / Postgrest Mocking
- Test Double Taxonomy

## Getting started

Start with these pages when choosing a mocking approach:

- Test Double Taxonomy
- Stubs Vs Mocks
- Mocking Order
- [Non-Factory `vi.mock` Pattern](./vitest-non-factory-mock.md)
- [`vi.mock()` vs `vi.spyOn()`](./vitest-mock-vs-spyon.md)
- [`vi.doMock()` — Runtime Exception Path](./vitest-domock.md)
- [Supabase / Postgrest Mocking](./vitest-supabase-postgrest-mocking.md)
- [Shared Mock Helper Infrastructure](./vitest-shared-mock-helper-infrastructure.md)
- [`forceCast` and `installStore`](./vitest-forceCast-and-the-installStore-selector-dispatch-pattern.md)
- [Setup: Mock External Modules](./vitest-api-handler-mock-setup.md)
- Choose One Mocking Seam

## Reference

### Case 1: Module-level initialization

Use a factory when a mocked function is called during another module's top-level initialization.
This happens at import time, before a test can call `mockReturnValue`.

```typescript
// api-client.ts (source module under test)
import { getConfig } from "./config";

// Runs ONCE at import time — not inside any function
export const BASE_URL = getConfig().apiUrl;
```

```typescript
// api-client.test.ts
// ❌ Non-factory fails: auto-mock returns undefined, BASE_URL is already frozen
vi.mock("./config");
const mockGetConfig = vi.mocked(getConfig);
// Too late — api-client already ran getConfig() during import
mockGetConfig.mockReturnValue({ apiUrl: "https://test.example.com" });

// ✅ Factory works: runs at hoist time, before api-client.ts initializes
vi.mock("./config", () => ({
	getConfig: vi.fn().mockReturnValue({ apiUrl: "https://test.example.com" }),
}));
import { fetchData } from "./api-client";
import { getConfig } from "./config";

const mockGetConfig = vi.mocked(getConfig);
```

### Case 2: Auto-mock cannot produce a spy-able export

Vitest's auto-mock replaces exports with `vi.fn()`. This works for plain functions and hooks, but
fails for exports that are not plain functions, such as `forwardRef` components, class instances,
or re-exports from complex package internals.

```typescript
// ❌ Auto-mock fails: Link is a forwardRef component, not a plain function.
vi.mock("react-router-dom");
vi.mocked(Link).mockImplementation((props) => <a href={props.to}>{props.children}</a>);

// ✅ PREFERRED: Use MemoryRouter instead of mocking react-router-dom.
// Wrap your component in MemoryRouter during render — this is more realistic and matches
// the repo's established pattern (see Navigation.test.tsx).
import { MemoryRouter } from "react-router-dom";

render(
	<MemoryRouter>
		<YourComponent />
	</MemoryRouter>,
);

// 🟡 Factory fallback: only if MemoryRouter cannot be used for your specific test scenario.
// The comment explaining WHY the factory is needed is required.
vi.mock("react-router-dom", () => ({
	Link: (props: React.ComponentProps<typeof Link>) => (
		<a href={props.to as string}>{props.children}</a>
	),
}));
```

**How to tell which case you're in:** For react-router-dom components like Link, always prefer
wrapping in `<MemoryRouter>` first. Only use factory mocking if MemoryRouter doesn't work for your
specific test scenario (rare).

### Default-export function components (app source)

When the SUT imports a **default-export function component** from this repo (not `forwardRef` from a
dependency), register the module with a bare `vi.mock("path")`, import that default binding, and
set the stub with `vi.mocked(Component).mockImplementation(...)` inside each `it` (or a small helper
the test calls after any `vi.resetAllMocks()`). Do **not** use a `vi.mock("path", () => ({ default:
... }))` factory for this case.

```ts
import EventLibraryCard from "@/react/event-library/card/EventLibraryCard";

vi.mock("@/react/event-library/card/EventLibraryCard");

it("isolates TagView from EventLibraryCard", () => {
	vi.mocked(EventLibraryCard).mockImplementation(({ entry }) => (
		<div data-testid={`event-card-${entry.event_id}`} />
	));
	// render the system under test …
});
```

If the test calls `vi.resetAllMocks()` first, re-apply `mockImplementation` afterward; reset clears
configured implementations.

Use `vi.spyOn()` only when you intentionally want to keep the real module implementation and patch
just one property for a test:

```ts
const mod = await import("@/shared/utils/date/clientLocalDateToUtcTimestamp");
vi.spyOn(mod, "default").mockReturnValue("2026-01-01T00:00:00Z");
```

| Situation                                             | Pattern                              |
| ----------------------------------------------------- | ------------------------------------ |
| Imported collaborator / repeated control across tests | `vi.mock("path")` + `vi.mocked(...)` |
| Advanced module-shape override                        | `vi.mock("path", factory)`           |
| One-off partial override on a stable object reference | `vi.spyOn(object, "method")`         |

Use `vi.spyOn()` as an escape hatch, not the baseline pattern.

## See also

- Authoring index
- Repo hub: `docs/testing/vitest-test-mock-best-practices.md`