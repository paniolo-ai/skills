---
source-slug: typescript-all-if-else-branches-contain-same-code-shared-error-check
source-hash: 8c05abe27566de9aa9f4e9295cb44c367197909228e656389c613e7e17c43fcd
bundled: 2026-10-17
title: Shared error handling across branches
type: concept
tags:
- authoring
- typescript
- lint
updated: 2026-07-24
---

# Shared error handling across branches

When every branch of an `if`/`else` ends in the same error-handling code, the linter flags the duplication. Throw inside `try` so one `catch` handles the shared error path:

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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
