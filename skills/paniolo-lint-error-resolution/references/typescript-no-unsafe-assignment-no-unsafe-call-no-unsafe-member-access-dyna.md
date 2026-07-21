---
source-wiki: sharp-shooter-wiki
source-slug: typescript-no-unsafe-assignment-no-unsafe-call-no-unsafe-member-access-dyna
source-hash: d51d35ebee317b7a757242c1ec8efd5de70424d382103079a05e18815a06b4e9
bundled: 2026-07-20
title: '`no-unsafe-assignment` / dynamic tables'
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

# `no-unsafe-assignment` / dynamic tables

Never cast the Supabase client to `any`. Use a typed branch per table:

```typescript
if (itemType === "song") {
	const result =
		yield *
		$(
			Effect.tryPromise({
				try: () => client.from("song_public").select("user_id").eq("song_id", itemId).single(),
				catch: (error) =>
					new DatabaseError({ message: extractErrorMessage(error, "Failed to fetch song") }),
			}),
		);
	return result.data.user_id;
}
```

## See also

- TypeScript lint (authoring) index