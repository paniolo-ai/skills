---
source-slug: store-composition
source-hash: 72816d8569b092877178e3691592e84c426caf0e48c9969e3cdfeedc60c60630
bundled: 2026-10-17
title: Store Composition (Multiple Stores)
type: concept
tags:
- authoring
- zustand
- react
- client
- state
updated: 2026-06-18
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
