---
source-slug: typescript-isolateddeclarations-explicit-type-annotations-on-exports
source-hash: 2cdfaa4842ef6dedfedf96d5072b5b971c7b3931a57d3010c18771fe4db032df
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
