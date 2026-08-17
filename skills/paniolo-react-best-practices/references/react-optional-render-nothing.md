---
source-wiki: sharp-shooter-wiki
source-slug: react-optional-render-nothing
source-hash: eafa6603242e7e829dfa3279cd908349b0ead4b01685766a6abb61dbf5a5ac57
bundled: 2026-08-16
title: Optional “render nothing” return
type: concept
tags:
- authoring
- react
- client
updated: 2026-06-18
---

# Optional “render nothing” return

Prefer **`return undefined`** with a **`ReactElement | undefined`** annotation when a component
sometimes renders nothing (invalid id, feature off, etc.), rather than **`return null`** with
**`ReactElement | null`**.

React treats both the same at runtime, but some lint configurations discourage `null` literals; using
`undefined` avoids disable comments and matches optional/absent semantics elsewhere.

```tsx
export default function SongSections({ songId }: SongSectionsProps): ReactElement | undefined {
	if (songId.trim() === "") {
		return undefined;
	}
	return <section>...</section>;
}
```

## See also

- React (authoring) index
