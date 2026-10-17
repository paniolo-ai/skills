---
source-slug: typescript-all-if-else-branches-contain-same-code-shared-error-check
source-hash: eca188197bb0ed18c8d188909ee6ade3db6a1e01110c880760c42f38f2442b5e
bundled: 2026-10-16
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
