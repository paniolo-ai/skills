---
source-slug: typescript-no-unsafe-type-assertion-request-validation
source-hash: 651707602f1b7a0b9439a1a2ab15893e4b401372c136ccf3d3a5e5f497a999a9
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
