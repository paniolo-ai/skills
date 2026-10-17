---
source-slug: persist-middleware
source-hash: cfeea42689307153ec2795bdc4dd990719f9ca64782d4ae5b951c9dbe3a5fa46
bundled: 2026-10-17
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

Use `persist` to automatically save state to `localStorage` and restore it on page reload. Use `partialize` to persist only specific fields:

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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
