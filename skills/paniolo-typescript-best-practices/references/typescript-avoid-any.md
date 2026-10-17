---
source-slug: typescript-avoid-any
source-hash: bfb3b86ff25ab90e79f2817ad22cba434fd4b8d77a2464865f5be8502a46ea6f
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
