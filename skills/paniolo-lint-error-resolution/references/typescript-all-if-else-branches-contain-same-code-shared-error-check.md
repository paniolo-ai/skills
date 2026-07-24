---
source-wiki: sharp-shooter-wiki
source-slug: typescript-all-if-else-branches-contain-same-code-shared-error-check
source-hash: 3d4839bba8e4e038836bb174bfd673f0bde618d79b23ac87009420f17b59955a
bundled: 2026-07-24
title: Shared error handling across branches
type: concept
tags:
- authoring
- typescript
- lint
updated: 2026-07-24
---

# Shared error handling across branches

When every branch of an `if`/`else` ends in the same error-handling code, the linter
flags the duplication. Throw inside `try` so one `catch` handles the shared error path:

```typescript
yield* $(Effect.tryPromise({
	try: async () => {
		const result = await client.from("my_table").insert([{ ... }]);
		if (result.error) throw result.error;
	},
	catch: (error: unknown) =>
		new DatabaseError({ message: extractErrorMessage(error, "Failed to insert") }),
}));
```

## See also

- TypeScript lint (authoring) index