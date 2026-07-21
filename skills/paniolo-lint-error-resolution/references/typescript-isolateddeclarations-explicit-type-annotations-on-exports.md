---
source-wiki: sharp-shooter-wiki
source-slug: typescript-isolateddeclarations-explicit-type-annotations-on-exports
source-hash: 9f26ea8247bc8403709653ada1a5dea8a0a0706e30ad50fe31d7a5b1c8bae6d6
bundled: 2026-07-20
title: '`--isolatedDeclarations` — explicit type annotations on exports'
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

# `--isolatedDeclarations` — explicit type annotations on exports

```typescript
export const mySchema: Schema.Schema<"a" | "b" | "c", "a" | "b" | "c"> = Schema.Literal(
	"a",
	"b",
	"c",
);
```

## See also

- TypeScript lint (authoring) index