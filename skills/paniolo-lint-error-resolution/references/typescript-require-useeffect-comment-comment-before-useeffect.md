---
source-wiki: sharp-shooter-wiki
source-slug: typescript-require-useeffect-comment-comment-before-useeffect
source-hash: 3c2100d683bbc913e2be4264ec4fc46d6faa665243215a302ad592c928c2f4be
bundled: 2026-08-01
title: '`require-useeffect-comment` — comment before `useEffect`'
type: concept
tags:
- authoring
- typescript
- lint
updated: 2026-06-18
---

# `require-useeffect-comment` — comment before `useEffect`

```typescript
// Fetch data on mount and re-fetch when the location changes.
useEffect(() => {
	void fetchData();
}, [fetchData]);
```

- No `useCallback` / `useMemo` — React Compiler project (see `react-best-practices` skill).
- No lint-disable in test files.

## See also

- TypeScript lint (authoring) index