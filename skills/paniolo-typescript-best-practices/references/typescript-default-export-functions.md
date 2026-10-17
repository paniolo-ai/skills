---
source-slug: typescript-default-export-functions
source-hash: e1b80ee320921c3b81ea728b56d9b95fc0aeda39849b458adeacb781171c66af
bundled: 2026-10-17
title: Default Export Functions
type: concept
tags:
- authoring
- typescript
updated: 2026-06-18
---

# Default Export Functions

Default-exported function modules should put the default export on the declaration itself:

```typescript
// ✅ Preferred: declaration and default export stay together
export default function formatSongTitle(title: string): string {
	return title.trim();
}

// ❌ Avoid: trailing default export for a declared function
function formatSongTitle(title: string): string {
	return title.trim();
}

export default formatSongTitle;
```

The same applies to async functions:

```typescript
export default async function fetchSongTitle(id: string): Promise<string> {
	return id;
}
```

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
