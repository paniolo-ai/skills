---
source-wiki: sharp-shooter-wiki
source-slug: typescript-effect-throwing-instead-of-effectfail
source-hash: 7d7e822ced2d225431451eff83b2e0fd9e120e44f35b2852d1bffe1e373aa20a
bundled: 2026-07-20
title: Throwing Instead of Effect.fail
type: concept
tags:
- authoring
- effect-ts
- effect
- typescript
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
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

## See also

- Effect-TS (authoring) index