---
source-slug: typescript-effect-defining-tagged-errors
source-hash: b141650f05c7df4f53ee225740e15237af617a4494196d73d95431560f6b92eb
bundled: 2026-10-16
title: Defining Tagged Errors
type: concept
tags:
- authoring
- effect-ts
- effect
- typescript
updated: 2026-06-18
---

# Defining Tagged Errors

Use `Data.TaggedError` for all API errors. The tag string is used by `Effect.catchTag` and enables discriminated union pattern matching at compile time.

```typescript
// api/src/api-errors.ts
import { Data } from "effect";

export class ValidationError extends Data.TaggedError("ValidationError") {
	constructor(readonly message: string) {
		super();
	}
}

export class NotFoundError extends Data.TaggedError("NotFoundError") {
	constructor(
		readonly resource: string,
		readonly id: string,
	) {
		super();
	}
}

export class DatabaseError extends Data.TaggedError("DatabaseError") {
	constructor(readonly message: string) {
		super();
	}
}

export class AuthenticationError extends Data.TaggedError("AuthenticationError") {
	constructor(readonly message: string) {
		super();
	}
}
```

**Why:** Structured errors give you compile-time guarantees about what can fail and enable
`Effect.catchTag` for targeted recovery. They also produce actionable error messages instead
of generic `"Something went wrong"` strings.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
