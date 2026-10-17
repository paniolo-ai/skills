---
source-slug: typescript-effect-mixing-promises-and-effects
source-hash: d721132d446eb095c6dd829c5405fdf1334b28597cdc3c0aa216baa36185fef7
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
