---
source-slug: typescript-effect-throwing-instead-of-effectfail
source-hash: c5ec1930f9d8fabbe871357b918c1aa7cd5d0532c5d096a82a24346cfbf02baa
bundled: 2026-10-17
title: Throwing Instead of Effect.fail
type: concept
tags:
- authoring
- effect-ts
- effect
- typescript
updated: 2026-06-18
---

# Throwing Instead of Effect.fail

Thrown exceptions inside `Effect.gen` bypass the error channel and produce untyped `Die` defects.

```typescript
// ❌ Wrong: throw inside Effect.gen
if (!record) {
	throw new NotFoundError({ resource: "Record", id });
}

// ✅ Correct: Effect.fail keeps the error typed
if (!record) {
	yield * Effect.fail(new NotFoundError({ resource: "Record", id }));
}
```

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
