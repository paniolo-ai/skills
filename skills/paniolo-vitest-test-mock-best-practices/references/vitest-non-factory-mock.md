---
source-wiki: sharp-shooter-wiki
source-slug: vitest-non-factory-mock
source-hash: 3b1c892adee1257dfe6968b01c0494dbf77f43fed8368c06485f06ae6ad846d4
bundled: 2026-07-20
title: Non-Factory `vi.mock` Pattern (Required)
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

# Non-Factory `vi.mock` Pattern (Required)

Use single-argument `vi.mock("path")` as the default. Avoid the factory pattern
(`vi.mock("path", () => {...})`) except when the non-factory approach cannot express the required
setup. Configure behavior with `vi.mocked(...)` inside each `it` block (or from a small helper each
test calls first). Do not use `beforeEach`/`afterEach`/`beforeAll`/`afterAll` in `*.test.*` files;
`jest/no-hooks` forbids lifecycle hooks—see [Avoid lifecycle hooks](#avoid-lifecycle-hooks). This
keeps mocks predictable in ESM-heavy code and avoids brittle hand-built module shapes.

**Preferred - non-factory mock registration + per-test `vi.mocked(...)`:**

```ts
import fetchEventCommunities from "@/react/event/fetch/fetchEventCommunities";
import subscribeToCommunityEventByEvent from "@/react/event/subscribe/subscribeToCommunityEventByEvent";

vi.mock("@/react/event/fetch/fetchEventCommunities");
vi.mock("@/react/event/subscribe/subscribeToCommunityEventByEvent");

it("uses mocked dependencies", () => {
	vi.mocked(fetchEventCommunities).mockReturnValue(Effect.succeed([]));
	vi.mocked(subscribeToCommunityEventByEvent).mockReturnValue(Effect.succeed(() => undefined));
});
```

Use a factory only when the non-factory pattern cannot express the setup. When a factory is
unavoidable, keep it minimal and typed.

Treat `vi.importActual` as a code smell by default in this repo. In most tests, if you need a
mocked dependency, prefer non-factory `vi.mock("path")` + `vi.mocked(...)` and avoid partial
module merging.

## Avoid lifecycle hooks

Do not use `beforeEach`, `afterEach`, `beforeAll`, or `afterAll` in `*.test.*` files.
Configure mocks inside each `it` block (or a helper the test calls first) so setup stays local
and `jest/no-hooks` stays green.

## See also

- Vitest mocking (authoring) index