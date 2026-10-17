---
source-slug: basic-selector-pattern
source-hash: f16f5997d57743e5b7348f5d9e61f82a9e17d7ae53d2333a5b9460a5be0240a0
bundled: 2026-10-16
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
