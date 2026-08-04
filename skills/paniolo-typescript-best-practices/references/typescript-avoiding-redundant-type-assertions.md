---
source-wiki: sharp-shooter-wiki
source-slug: typescript-avoiding-redundant-type-assertions
source-hash: dfd3db4cca18e83b016444dacc24d8273a6e7e88986ea2ce7dc60e304b3f96ea
bundled: 2026-08-01
title: Avoiding Redundant Type Assertions
type: concept
tags:
- authoring
- typescript
updated: 2026-06-18
---

# Avoiding Redundant Type Assertions

Prefer an explicit type annotation on the receiving variable over an inline `as` cast. The
annotation is sufficient and does not suppress type errors the way `as` can.

```typescript
// ❌ BAD: redundant — `raw` annotation already widens any to unknown
const raw: unknown = await (response.json() as Promise<unknown>);

// ✅ GOOD: the explicit variable annotation handles it
const raw: unknown = await response.json();

// ❌ BAD: same issue inside Effect.tryPromise
Effect.tryPromise({ try: () => response.json() as Promise<unknown>, catch: ... })

// ✅ GOOD
const raw: unknown = yield* $(Effect.tryPromise({ try: () => response.json(), catch: ... }))
```

Reserve `as SomeType` for genuine narrowings where TypeScript cannot infer the type (e.g. after a
runtime discriminant check).

## See also

- TypeScript (authoring) index