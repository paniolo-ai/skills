---
source-wiki: sharp-shooter-wiki
source-slug: react-compiler
source-hash: c3efefa30544cb40cf356840f74f70e1ab743136495cee29a15313bdad2773b3
bundled: 2026-08-01
title: React Compiler
type: concept
tags:
- authoring
- react
- client
updated: 2026-06-18
---

# React Compiler

This project uses the React Compiler, which automatically optimizes components and hooks. **NEVER
use `useCallback` or `useMemo`** to stabilize props, dependency lists, or derived values — the
compiler handles all identity stability and memoization automatically. Manual memoization
is strictly forbidden to avoid conflicts and redundant overhead:

```tsx
// ❌ NEVER USE — compiler handles this automatically
const handleClick = useCallback(() => { doSomething(value); }, [value]);
const expensiveValue = useMemo(() => computeExpensiveValue(data), [data]);
export const MyComponent = memo(({ prop }) => { ... });

// ✅ Preferred — let the compiler optimize (same pattern inside hooks)
function handleClick(): void { doSomething(value); }
const expensiveValue = computeExpensiveValue(data);
export function MyComponent({ prop }: MyComponentProps): ReactElement { ... }
```

**Rare exceptions:** confirmed and measured performance issues the compiler doesn't optimize;
interfacing with non-React code. When adding manual memoization (e.g. `memo`) for an exception,
include a comment explaining why and link to the performance trace or issue. **`useCallback`
and `useMemo` remain forbidden even in these cases.**

## See also

- React (authoring) index
