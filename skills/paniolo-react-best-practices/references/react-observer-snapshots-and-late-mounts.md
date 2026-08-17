---
source-wiki: sharp-shooter-wiki
source-slug: react-observer-snapshots-and-late-mounts
source-hash: ece92a159ca842088dca034598d815b5c21e31c513eee7d7377ee4fb10a1cc9a
bundled: 2026-08-15
title: Observer Snapshots and Late Mounts
type: concept
tags:
- authoring
- react
- client
- dom
updated: 2026-08-15
---

# Observer Snapshots and Late Mounts

`IntersectionObserver` has two properties that read as conveniences and behave as traps.

## 1. Entry geometry is a snapshot, not live

`IntersectionObserverEntry.boundingClientRect` is captured **at the moment the threshold was
crossed**. Hold the entry and read its rect later and you get where the element *was*, not where it
is. After any scroll the answer is wrong, and wrong in a way that looks plausible.

```ts
// ❌ Ranking cached entries by a rect captured at threshold crossing
const topmost = [...entries.values()].sort(
	(a, b) => a.boundingClientRect.top - b.boundingClientRect.top,
)[0];

// ✅ Use the observer to decide *which* elements are candidates,
//    then measure them now
const topmost = pickTopVisible(
	[...observed].map((element) => ({ element, top: element.getBoundingClientRect().top })),
);
```

Treat the observer as a **notification that the set of visible elements changed**, and measure with
`getBoundingClientRect()` at the moment you need a position.

## 2. An observer only watches what existed when it was constructed

`observe()` takes specific nodes. A list that mounts its items lazily — windowing, virtualization,
pagination, infinite scroll — adds nodes the observer has never been told about, and they are
silently never reported.

The symptom is that tracking works for the first screenful and then stops, which reads like a
threshold or `rootMargin` problem rather than a missing `observe()` call.

Re-sync the observed set when the subtree changes:

```ts
useEffect(() => {
	const container = containerRef.current;
	if (container === null) {
		return;
	}

	const observer = new IntersectionObserver(handleEntries, { root: container });

	/** Observe every current anchor, including ones mounted after setup. */
	function syncObserved(): void {
		observer.disconnect();
		for (const anchor of container.querySelectorAll("[data-anchor]")) {
			observer.observe(anchor);
		}
	}

	syncObserved();
	const mutations = new MutationObserver(syncObserved);
	mutations.observe(container, { childList: true, subtree: true });

	return (): void => {
		observer.disconnect();
		mutations.disconnect();
	};
}, [handleEntries]);
```

## Compiler note

Holding the caller's callback in a ref to keep it out of the dependency array is the usual way to
stop this effect rebuilding every render. Under the React Compiler, assign that ref inside
`useLayoutEffect` — a render-phase write is a build error. See
Compiler-rejected render side effects.

## See also

- Windowed lists need an estimated size
- useEffect Rules
- React (authoring) index
