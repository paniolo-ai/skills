---
source-slug: basic-selector-pattern
source-hash: 70d646c1cf4aa935aa0fe0e463c8da78e9e16f9a72f63cd367f861151eee4c17
bundled: 2026-10-17
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

Always select only the fields you need. Selecting the whole store causes a re-render whenever **any** field changes:

```typescript
// ❌ Whole store — re-renders on any state change
const state = useCounterStore();

// ✅ Select only what you need
const count = useCounterStore((state) => state.count);
const increment = useCounterStore((state) => state.increment);
```

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
