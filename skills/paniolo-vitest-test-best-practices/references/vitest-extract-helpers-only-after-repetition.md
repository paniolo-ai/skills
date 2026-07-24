---
source-wiki: sharp-shooter-wiki
source-slug: vitest-extract-helpers-only-after-repetition
source-hash: 4017353e0d828d2a62ee87d6d2e3dd430d9bc5116754f6780078910aab846309
bundled: 2026-07-24
title: Extract Helpers Only After Repetition
type: concept
tags:
- authoring
- vitest
- testing
updated: 2026-07-14
---

# Extract Helpers Only After Repetition

Keep one-off setup inline. Extract a local typed factory when the same complete prop, callback,
or mock bundle repeats two or more times in a spec. This keeps each assertion focused on the
meaningful override without hiding the test's behavior behind a general-purpose helper.

## Local factories for repeated test setup

Use a descriptive local factory such as `makeProps`, `makeHookProps`, `makeCallbacks`, or
`makeStoreState`. It should return the real type, supply valid defaults, and accept `Partial<T>`
overrides:

```ts
function makeProps(overrides: Partial<ComponentProps<typeof Player>> = {}) {
	return {
		onClose: makeNoopFn(),
		playing: false,
		trackId: "track-1",
		...overrides,
	};
}

const props = makeProps({ playing: true });
render(<Player {...props} />);
```

- Create the fixture once, then retain and assert against the same spies. Do not call a factory
  again just to obtain a handler; that produces a different mock and makes observability unclear.
- Keep a helper local while it serves one spec. Promote it to a colocated `*.test-util.ts` only
  after multiple specs need the same domain fixture. Reuse an existing canonical factory before
  creating either kind of helper.
- Use `makeNoopFn()` for a behaviorally irrelevant callback. It is intentionally a no-argument,
  void-returning mock, which is assignable to ordinary void callbacks without repeating their
  signature. Use `vi.fn<ActualSignature>()` when the test asserts calls or configures its return
  value, implementation, or resolved/rejected behavior. Use a local structural adapter for
  overloaded callback types such as Zustand `Set`.
- Do not extract one-off setup or use a factory to conceal the values that define the assertion.
  Tests should still make their distinguishing inputs explicit.

## See also

- Vitest unit testing (authoring) index
- Prefer shared `make*` helpers