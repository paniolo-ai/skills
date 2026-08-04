---
source-wiki: sharp-shooter-wiki
source-slug: persist-middleware
source-hash: dfb595795b0b972dd520b9270ba0bce5f0ca9bf0fae0178c5810e07dbc18898d
bundled: 2026-08-01
title: Persist Middleware
type: concept
tags:
- authoring
- zustand
- react
- client
- state
updated: 2026-06-18
---

# Persist Middleware

Use `persist` to automatically save state to `localStorage` and restore it on page reload. Use
`partialize` to persist only specific fields:

```typescript
import { persist } from "zustand/middleware";

export const useThemeStore = create<ThemeState>()(
	persist(
		(set) => ({
			theme: "light",
			toggleTheme: () => set((state) => ({ theme: state.theme === "light" ? "dark" : "light" })),
		}),
		{
			name: "theme-storage", // localStorage key
			partialize: (state) => ({ theme: state.theme }), // only persist theme
		},
	),
);
```

## See also

- Zustand (authoring) index