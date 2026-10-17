---
source-slug: react-optional-render-nothing
source-hash: e4a93cd9b070d484c87bec0b456d634e477861a1c9d0902e6a3a22b939d92a9d
bundled: 2026-10-16
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
