---
source-wiki: sharp-shooter-wiki
source-slug: typescript-default-export-functions
source-hash: 75e449dc45aa212b0a197c00e0532218f555925dbfcbb8e61570865e2a1158b6
bundled: 2026-07-20
---
---
title: "Default Export Functions"
type: concept
tags: [authoring, typescript]
updated: 2026-06-18
sources:
  - raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
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

## See also

- TypeScript (authoring) index
