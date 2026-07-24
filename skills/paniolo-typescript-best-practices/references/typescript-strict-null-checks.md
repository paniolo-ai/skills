---
source-wiki: sharp-shooter-wiki
source-slug: typescript-strict-null-checks
source-hash: 726980e5e897d1890ba460b211b587105029ceae4a2f5576449e2fa7b1b5b93a
bundled: 2026-07-24
title: Strict Null Checks
type: concept
tags:
- authoring
- typescript
updated: 2026-06-18
---

# Strict Null Checks

`strictNullChecks` is enabled. Always handle `null` and `undefined` explicitly:

```typescript
// ❌ Avoid: assuming value is defined
const getName = (user: User | undefined) => user.name; // TypeScript error

// ✅ Preferred: explicit guard
const getName = (user: User | undefined): string => {
	if (user === undefined) {
		return "Unknown";
	}
	return user.name;
};

// ✅ Preferred: optional chaining + nullish coalescing
const getName = (user: User | undefined): string => user?.name ?? "Unknown";
```

## See also

- TypeScript (authoring) index