---
source-slug: devtools
source-hash: 72ab56a7adba90eace1eb4aa5c4e56d10b1fce223e7c54d8c4c301a24d65f238
bundled: 2026-10-16
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

Wrap the store with `devtools` to get action history and time-travel debugging in the Redux DevTools browser extension:

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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
