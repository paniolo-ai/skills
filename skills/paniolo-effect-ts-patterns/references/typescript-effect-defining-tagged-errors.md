---
source-wiki: sharp-shooter-wiki
source-slug: typescript-effect-defining-tagged-errors
source-hash: f66960132639ca92aed3f3941c599b5aff9d99d46e94ea23dcb5e3030c47f5eb
bundled: 2026-08-01
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

Use `Data.TaggedError` for all API errors. The tag string is used by `Effect.catchTag` and
enables discriminated union pattern matching at compile time.

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

## See also

- Effect-TS (authoring) index
