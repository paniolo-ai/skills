---
source-slug: react-optional-render-nothing
source-hash: 705c8b3a881c8774cefa57ae2b40e2647a239a2ace6b6306031ca903f15ae32e
bundled: 2026-10-17
title: Optional “render nothing” return
type: concept
tags:
- authoring
- react
- client
updated: 2026-06-18
---

# Optional “render nothing” return

Prefer **`return undefined`** with a **`ReactElement | undefined`** annotation when a component sometimes renders nothing (invalid id, feature off, etc.), rather than **`return null`** with **`ReactElement | null`**.

React treats both the same at runtime, but some lint configurations discourage `null` literals; using `undefined` avoids disable comments and matches optional/absent semantics elsewhere.

```tsx
export default function SongSections({ songId }: SongSectionsProps): ReactElement | undefined {
	if (songId.trim() === "") {
		return undefined;
	}
	return <section>...</section>;
}
```

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
