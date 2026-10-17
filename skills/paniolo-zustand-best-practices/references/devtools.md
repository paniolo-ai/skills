---
source-slug: devtools
source-hash: 7be95945f7ba8dc46bc8cc59b6641c832f9683d51e62d001b9c621480f17ffe4
bundled: 2026-10-17
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
