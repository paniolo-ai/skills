---
source-slug: typescript-avoiding-redundant-type-assertions
source-hash: d1a3dec1a9204239ec3b8fb6eb39bc4f225240c22a0fa0a967d0075db31c0e06
bundled: 2026-10-17
title: Avoiding Redundant Type Assertions
type: concept
tags:
- authoring
- typescript
updated: 2026-06-18
---

# Avoiding Redundant Type Assertions

Prefer an explicit type annotation on the receiving variable over an inline `as` cast. The annotation is sufficient and does not suppress type errors the way `as` can.

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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
