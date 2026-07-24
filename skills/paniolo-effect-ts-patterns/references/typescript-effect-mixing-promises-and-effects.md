---
source-wiki: sharp-shooter-wiki
source-slug: typescript-effect-mixing-promises-and-effects
source-hash: 0af9a8c396656cf0146a6cab4dc390020214e6fa5eece48ba9c1019f241841a5
bundled: 2026-07-24
title: Mixing Promises and Effects
type: concept
tags:
- authoring
- effect-ts
- effect
- typescript
updated: 2026-06-18
---

# Mixing Promises and Effects

Do not mix `await` and `yield*` inside the same `Effect.gen` block.

```typescript
// ❌ Wrong: mixing await and yield*
const myEffect = Effect.gen(function* () {
	const data = await somePromise; // breaks the Effect fiber
	const result = yield* service.doWork();
});

// ✅ Correct: wrap the Promise in Effect.tryPromise
const myEffect = Effect.gen(function* () {
	const data = yield* Effect.tryPromise({
		try: () => somePromise,
		catch: (e) => new ApiError({ message: String(e) }),
	});
	const result = yield* service.doWork();
});
```

## See also

- Effect-TS (authoring) index