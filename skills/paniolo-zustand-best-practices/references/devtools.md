---
source-wiki: sharp-shooter-wiki
source-slug: devtools
source-hash: 2d6894fb0867f68d5b402521c775db11b8050e0b44e47afb254a4e8171d05457
bundled: 2026-07-24
title: DevTools
type: concept
tags:
- authoring
- zustand
- react
- client
- state
updated: 2026-06-18
---

# DevTools

Wrap the store with `devtools` to get action history and time-travel debugging in the Redux DevTools
browser extension:

```typescript
import { devtools } from "zustand/middleware";

export const useCounterStore = create<CounterState>()(
	devtools(
		(set) => ({
			count: 0,
			increment: () => set((state) => ({ count: state.count + 1 })),
		}),
		{ name: "counter-store" },
	),
);
```

## See also

- Zustand (authoring) index