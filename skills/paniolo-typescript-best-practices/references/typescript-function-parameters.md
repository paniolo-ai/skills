---
source-wiki: sharp-shooter-wiki
source-slug: typescript-function-parameters
source-hash: c83006f2082ecd20aa6f9c49a78d374661d9605781a022d6c064bf424e83401e
bundled: 2026-07-20
title: Function Parameters
type: concept
tags:
- authoring
- typescript
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Function Parameters

Prefer destructured parameters directly in the function signature over a single `params` object
pattern. This is more readable, consistent with destructuring at call sites, and aligns with the
linter's max-parameters rule.

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

## See also

- TypeScript (authoring) index