---
source-slug: typescript-require-useeffect-comment-comment-before-useeffect
source-hash: 16e02ebe9ccc507dd3b3da92e020b5006717a1e7ba616ad9335437fa35ce2120
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
