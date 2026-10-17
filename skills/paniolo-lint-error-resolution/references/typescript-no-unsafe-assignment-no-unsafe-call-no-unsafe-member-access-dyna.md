---
source-slug: typescript-no-unsafe-assignment-no-unsafe-call-no-unsafe-member-access-dyna
source-hash: f58e7fa005a8bc0c78a2ec3ebb7387d7d27b548d4f9a95d234d3f5541e80ea50
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
