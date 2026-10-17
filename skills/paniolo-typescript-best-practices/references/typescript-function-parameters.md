---
source-slug: typescript-function-parameters
source-hash: 861dbc57440d217503c28bbd66c1a2b3cb71d5e371783660da912440153dcfed
bundled: 2026-10-17
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
