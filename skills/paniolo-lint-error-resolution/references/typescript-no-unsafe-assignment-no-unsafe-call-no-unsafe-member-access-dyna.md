---
source-wiki: sharp-shooter-wiki
source-slug: typescript-no-unsafe-assignment-no-unsafe-call-no-unsafe-member-access-dyna
source-hash: 778631c73cf159664da49089365901f5bd85c7c3bffda192d220176e9a633fde
bundled: 2026-07-24
title: '`no-unsafe-assignment` / dynamic tables'
type: concept
tags:
- authoring
- typescript
- lint
updated: 2026-06-18
---

# `no-unsafe-assignment` / dynamic tables

Never cast the database client to `any`. Use a typed branch per table:

```typescript
if (itemType === "item") {
	const result =
		yield *
		$(
			Effect.tryPromise({
				try: () => client.from("item_public").select("user_id").eq("item_id", itemId).single(),
				catch: (error) =>
					new DatabaseError({ message: extractErrorMessage(error, "Failed to fetch item") }),
			}),
		);
	return result.data.user_id;
}
```

## See also

- TypeScript lint (authoring) index