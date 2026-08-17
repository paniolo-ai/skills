---
source-wiki: sharp-shooter-wiki
source-slug: react-compiler-no-render-side-effects
source-hash: da9fb8090de66b24c63bd55feec94fa61643dd582e5c6e39662c8a7cd167d9a7
bundled: 2026-08-15
title: Compiler-rejected render side effects
type: concept
tags:
- authoring
- react
- client
updated: 2026-08-15
---

# Compiler-rejected render side effects

Writing to a ref during render is a **`ReactCompilerError`**, not a warning. With
`panicThreshold: "all_errors"` it fails the dev build, so it cannot be discovered later:

```tsx
// ❌ ReactCompilerError: Ref values may not be written during render
function useTopVisibleElement({ onTopElement }: Options): void {
	const onTopElementRef = useRef(onTopElement);
	onTopElementRef.current = onTopElement; // ← render-phase write

	useEffect(() => { /* reads onTopElementRef.current */ }, []);
}

// ✅ Same latest-value behaviour, written in a commit-phase effect
function useTopVisibleElement({ onTopElement }: Options): void {
	const onTopElementRef = useRef(onTopElement);

	// Keep the ref pointing at the newest callback without re-running the
	// observer effect below, which would tear down and rebuild the observer.
	useLayoutEffect(() => {
		onTopElementRef.current = onTopElement;
	}, [onTopElement]);

	useEffect(() => { /* reads onTopElementRef.current */ }, []);
}
```

Use **`useLayoutEffect`**, not `useEffect`, for this. The ref must be current before any effect
that reads it runs; a plain `useEffect` can land after a sibling effect has already fired with the
stale value.

## Why the pattern shows up at all

The "latest callback in a ref" pattern exists to keep an expensive subscription — an
`IntersectionObserver`, a `MutationObserver`, an event listener — out of a dependency array. Putting
the callback in the deps instead is correct but tears the subscription down and rebuilds it on every
render, because [the compiler does not stabilise props you did not memoise](./react-compiler.md) and
memoisation is forbidden.

So the ordering is:

1. Can the effect just depend on the callback? Prefer that — it is simpler.
2. If rebuilding the subscription each render is too costly, use the ref, and assign it in
   `useLayoutEffect`.

## Other render-phase writes the compiler rejects

The rule is broader than refs: **render must be pure**. Mutating anything that outlives the render
is rejected or produces wrong output under the compiler's re-execution assumptions.

- Writing to a ref (above).
- Mutating a value captured from props or state — including `array.sort()` and `array.reverse()`,
  which mutate in place. Copy first: `[...items].sort(…)`.
- Assigning to a module-level `let` from render.
- Calling a store's setter during render.

## See also

- React Compiler
- Compiler-compatible control flow
- useEffect rules
