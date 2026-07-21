---
source-wiki: sharp-shooter-wiki
source-slug: schema-definition
source-hash: 0001ee54de2d7b6eeba57f18b5e9e9101dad0cb6bb2cc599e90ed64ab1d2612a
bundled: 2026-07-20
title: Schema Definition
type: concept
tags:
- authoring
- form
- react
- validation
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Schema Definition

Colocate the schema in a `<feature>FormSchema.ts` file next to the component:

```typescript
// react/src/song/SongFormSchema.ts
import { Schema } from "effect";

export const SongFormSchema = Schema.Struct({
	title: Schema.String.pipe(Schema.minLength(1)),
	artist: Schema.String.pipe(Schema.minLength(1)),
});

export type SongFormValues = Schema.Schema.Type<typeof SongFormSchema>;
```

## See also

- Forms (authoring) index