---
source-wiki: sharp-shooter-wiki
source-slug: typescript-no-unsafe-type-assertion-request-validation
source-hash: 5c94c6d7aed0c9c39dd63cf1f557f28ea42a30549f238f968a4ac2ccc91ddb60
bundled: 2026-07-20
title: '`no-unsafe-type-assertion` — request validation'
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

# `no-unsafe-type-assertion` — request validation

Never cast `request as Record<string, unknown>`. Use `decodeUnknownSyncOrThrow` with a schema:

```typescript
import decodeUnknownSyncOrThrow from "@/shared/validation/decodeUnknownSyncOrThrow";
import { mySchema } from "@/shared/validation/mySchemas";

export default function extractMyRequest(request: unknown): MyRequest {
	return decodeUnknownSyncOrThrow(mySchema, request);
}
```

## See also

- TypeScript lint (authoring) index