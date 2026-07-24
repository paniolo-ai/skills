---
source-wiki: sharp-shooter-wiki
source-slug: react-optional-render-nothing
source-hash: 942ccf90c4ad139db4d02473a3adb89c0227b838cace8165cd2752027ebd4609
bundled: 2026-07-24
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

React treats both the same at runtime, but this repo discourages `null` literals under lint; using
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