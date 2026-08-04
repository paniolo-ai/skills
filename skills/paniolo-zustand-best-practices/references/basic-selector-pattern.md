---
source-wiki: sharp-shooter-wiki
source-slug: basic-selector-pattern
source-hash: 727fef21842a2d06976e5a41bba61ef6d1402030b641193809de17777f914698
bundled: 2026-08-01
title: Basic Selector Pattern
type: concept
tags:
- authoring
- zustand
- react
- client
- state
updated: 2026-06-18
---

# Basic Selector Pattern

Always select only the fields you need. Selecting the whole store causes a re-render whenever
**any** field changes:

```typescript
// ❌ Whole store — re-renders on any state change
const state = useCounterStore();

// ✅ Select only what you need
const count = useCounterStore((state) => state.count);
const increment = useCounterStore((state) => state.increment);
```

## See also

- Zustand (authoring) index