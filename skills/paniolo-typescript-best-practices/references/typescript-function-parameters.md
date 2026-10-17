---
source-slug: typescript-function-parameters
source-hash: 95694fa9e4f7bda156f40db3a5f6d9024825000ec6d2012320e1bd7f50c6e923
bundled: 2026-10-16
title: Function Parameters
type: concept
tags:
- authoring
- typescript
updated: 2026-06-18
---

# Function Parameters

Prefer destructured parameters directly in the function signature over a single `params` object pattern. This is more readable, consistent with destructuring at call sites, and aligns with the linter's max-parameters rule.

```typescript
// ✅ Preferred: destructure directly in signature
function handleEvent({
	payload,
	currentEvent,
	setCurrentEvent,
	fetchEventBySlug,
}: Readonly<HandleEventParams>): Effect.Effect<void, Error> {
	// use payload, currentEvent, etc. directly
}

// ❌ Avoid: single params object with internal destructuring
function handleEvent(params: Readonly<HandleEventParams>): Effect.Effect<void, Error> {
	const { payload, currentEvent, setCurrentEvent, fetchEventBySlug } = params;
	// ...
}
```

**Why:** Direct destructuring makes parameters immediately visible at the function definition,
improves IDE autocomplete and documentation, and is more idiomatic in modern TypeScript. When the
linter flags too many parameters, this approach satisfies the max-parameters rule while keeping
code readable.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
