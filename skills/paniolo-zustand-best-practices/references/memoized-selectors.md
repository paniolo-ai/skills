---
source-slug: memoized-selectors
source-hash: c69f529a9fa2c9556cf95a6354f6730735bbc65a2133accc5ed5029ee25b2d19
bundled: 2026-10-17
title: Memoized Selectors
type: concept
tags:
- authoring
- zustand
- react
- client
- state
updated: 2026-06-18
---

# Memoized Selectors

For selectors that derive new arrays or objects, use `useShallow` to prevent unnecessary re-renders:

```typescript
import { useShallow } from "zustand/react/shallow";

// Without useShallow: new array reference on every render even if contents are equal
const titles = useTodoStore((state) => state.todos.map((t) => t.title));

// With useShallow: only re-renders when the derived array contents change
const titles = useTodoStore(useShallow((state) => state.todos.map((t) => t.title)));
```

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
