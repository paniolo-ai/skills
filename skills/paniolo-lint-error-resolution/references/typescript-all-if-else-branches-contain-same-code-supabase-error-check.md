---
source-wiki: sharp-shooter-wiki
source-slug: typescript-all-if-else-branches-contain-same-code-supabase-error-check
source-hash: e56fbb686c966550400de9678c49eb63c3696b2a2ff0098e7f42f5d8f4f65cad
bundled: 2026-07-20
title: Shared Supabase error handling
type: concept
tags:
- authoring
- typescript
- lint
- songshare
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Shared Supabase error handling

Throw inside `try` so one `catch` handles the shared error path:

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