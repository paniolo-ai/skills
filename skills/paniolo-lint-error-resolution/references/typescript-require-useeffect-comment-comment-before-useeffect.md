---
source-wiki: sharp-shooter-wiki
source-slug: typescript-require-useeffect-comment-comment-before-useeffect
source-hash: fef7171a8dac33fcb7406e14a88422f33c4197f4647c387448fe96e756162359
bundled: 2026-07-20
title: '`require-useeffect-comment` — comment before `useEffect`'
type: concept
tags:
- authoring
- typescript
- lint
- songshare
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
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