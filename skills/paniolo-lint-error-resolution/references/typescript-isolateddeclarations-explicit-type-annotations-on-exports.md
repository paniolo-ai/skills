---
source-wiki: sharp-shooter-wiki
source-slug: typescript-isolateddeclarations-explicit-type-annotations-on-exports
source-hash: f31decd3b819a978c8eff1898b06bc9a1a01b1783b1f1a94425e0d1257453e25
bundled: 2026-07-24
title: '`--isolatedDeclarations` — explicit type annotations on exports'
type: concept
tags:
- authoring
- typescript
- lint
updated: 2026-06-18
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