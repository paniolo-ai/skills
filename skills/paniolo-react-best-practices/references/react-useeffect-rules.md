---
source-wiki: sharp-shooter-wiki
source-slug: react-useeffect-rules
source-hash: 21be01b24853e663a328a4011854a14a872a927f3c086af08630cf9e15483cd9
bundled: 2026-07-20
title: useEffect Rules
type: concept
tags:
- authoring
- react
- client
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# useEffect Rules

**Always add a comment on the line directly above `useEffect` explaining why the effect exists:**

```tsx
// ❌ No comment — unclear why this effect runs
useEffect(() => {
	document.title = `Song: ${title}`;
}, [title]);

// ✅ Comment explains the reason
// Sync the browser tab title whenever the active song title changes
useEffect(() => {
	document.title = `Song: ${title}`;
}, [title]);
```

**Keep dependency arrays complete — do not suppress the exhaustive-deps lint rule:**

```tsx
// ❌ Suppressed dep lint hides a real bug
useEffect(() => {
	fetchSong(songId);
	// oxlint-disable-next-line react/exhaustive-deps
}, []);

// ✅ Complete deps — effect re-runs correctly when songId changes
// Fetch song data whenever the active song ID changes
useEffect(() => {
	fetchSong(songId);
}, [songId]);
```

If exhaustive-deps flags a dep you genuinely don't want, restructure the code (move stable values
outside the component, or capture in a ref). When restructuring isn't feasible (e.g., a
store-provided function that is recreated on every render), omit it and document the intent:

```tsx
// Sync subscriptions when the active song changes.
// storeAction is excluded: it is recreated each render but is referentially stable in practice.
// oxlint-disable-next-line react/exhaustive-deps
useEffect(() => {
	storeAction(songId);
}, [songId]);
```

**When depending on a collection** (e.g., a list of IDs), include a primitive key — a
sorted-joined string — rather than the full array object. Array references change on every render
even when the contents are the same:

```tsx
const songIdKey = [...songIds].sort().join(",");

// Re-run only when the set of IDs actually changes
useEffect(() => {
	loadSongs(songIds);
	// oxlint-disable-next-line react/exhaustive-deps
}, [songIdKey]);
```

**Clean up async effects with an `isMounted` flag to prevent state updates after unmount:**

```tsx
// Fetch song data whenever the active song ID changes
useEffect(() => {
	let isMounted = true;

	async function load(): Promise<void> {
		const data = await fetchSong(songId);
		if (isMounted) {
			setSong(data);
		}
	}

	void load();

	return () => {
		isMounted = false;
	};
}, [songId]);
```

## See also

- React (authoring) index