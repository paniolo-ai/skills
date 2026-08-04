---
source-wiki: sharp-shooter-wiki
source-slug: typescript-no-unsafe-type-assertion-request-validation
source-hash: a61add85dfee99a4a45b3c1ef49e04a6d90a6f88f5d6807472580fcbe424cf77
bundled: 2026-08-01
title: '`no-unsafe-type-assertion` — request validation'
type: concept
tags:
- authoring
- typescript
- lint
updated: 2026-06-18
---

# `no-unsafe-type-assertion` — request validation

Never cast `request as Record<string, unknown>`. Use `decodeUnknownSyncOrThrow` with a schema:

```typescript
import decodeUnknownSyncOrThrow from "@shared/validation/decodeUnknownSyncOrThrow";
import { mySchema } from "@shared/validation/mySchemas";

export default function extractMyRequest(request: unknown): MyRequest {
	return decodeUnknownSyncOrThrow(mySchema, request);
}
```

## See also

- TypeScript lint (authoring) index