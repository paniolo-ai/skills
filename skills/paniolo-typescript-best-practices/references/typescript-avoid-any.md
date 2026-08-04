---
source-wiki: sharp-shooter-wiki
source-slug: typescript-avoid-any
source-hash: 138e7963fbab074bc09358c30ccf40351a6a961e77bd8972dd6cecf46acb639b
bundled: 2026-08-01
title: Avoid `any`
type: concept
tags:
- authoring
- typescript
updated: 2026-06-18
---

# Avoid `any`

Never use `any`. Use `unknown` for values whose type is truly unknown, and narrow with type guards:

```typescript
// ❌ Avoid: any disables type checking
const process = (value: any) => value.toUpperCase();

// ✅ Preferred: unknown + type guard
const process = (value: unknown): string => {
	if (typeof value === "string") {
		return value.toUpperCase();
	}
	throw new Error("Expected string");
};

// ✅ Preferred: specific type
const process = (data: { value: string }): string => data.value;
```

## See also

- TypeScript (authoring) index