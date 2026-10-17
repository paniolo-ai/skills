---
source-slug: typescript-strict-null-checks
source-hash: 8f1d4cc43c4313252491a981c7815b06620ba97ab78cda610a78ddca72f30830
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
