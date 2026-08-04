---
source-wiki: sharp-shooter-wiki
source-slug: memoized-selectors
source-hash: 3a14e4bf6aa0467efdb577800515bdab79e5c9e8a57b91338a0bd2ce8e9046ef
bundled: 2026-08-01
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

## See also

- Zustand (authoring) index