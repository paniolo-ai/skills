---
source-wiki: sharp-shooter-wiki
source-slug: store-composition
source-hash: b88f13fbd67f83fca1bc346370ec6a2d20f33c07ecef2593418406373e5d2fd8
bundled: 2026-07-20
title: Store Composition (Multiple Stores)
type: concept
tags:
- authoring
- zustand
- react
- client
- state
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Store Composition (Multiple Stores)

Keep one store per domain. Components can subscribe to multiple stores:

```typescript
export const useAuthStore = create<AuthState>((set) => ({
  isSignedIn: false,
  user: null,
  signOut: () => set({ isSignedIn: false, user: null }),
}));

export const useThemeStore = create<ThemeState>((set) => ({
  theme: "light",
  toggleTheme: () =>
    set((state) => ({ theme: state.theme === "light" ? "dark" : "light" })),
}));

// Usage — subscribe to each independently
function Header(): ReactElement {
  const isSignedIn = useAuthStore((state) => state.isSignedIn);
  const theme = useThemeStore((state) => state.theme);
  return <header className={theme}>{isSignedIn ? "Welcome" : "Sign In"}</header>;
}
```

## See also

- Zustand (authoring) index