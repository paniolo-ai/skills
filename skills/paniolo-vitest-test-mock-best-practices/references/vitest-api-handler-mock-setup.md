---
source-wiki: sharp-shooter-wiki
source-slug: vitest-api-handler-mock-setup
source-hash: cf34532490a93eeed7d44a3fae4ca1f52cd60af93ef41cc433f97db7d3456c19
bundled: 2026-07-20
title: 'Setup: Mock External Modules'
type: concept
tags:
- authoring
- vitest
- testing
- mocking
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Setup: Mock External Modules

```typescript
import { createClient } from "@supabase/supabase-js";
import { Effect } from "effect";
import { describe, expect, it, vi } from "vitest";

import type { UserSessionData } from "@/shared/userSessionData";
import { AuthenticationError } from "@/api/api-errors";
import makeCtx from "@/api/hono/makeCtx.test-util";
import mockCreateSupabaseClient from "@/api/test-utils/mockCreateSupabaseClient.test-util";
import accountDelete from "./accountDelete";

vi.mock("@supabase/supabase-js");
vi.mock("@/api/cookie/buildClearCookieHeader");
vi.mock("@/api/user-session/getVerifiedSession");
vi.mock("@/api/csrf/verifySameOriginOrThrow");
vi.mock("@/api/csrf/verifyDoubleSubmitOrThrow");
```

Before writing new local stubs, check the **feature folder** for colocated `make*.test-util.ts`
first, then `api/src/test-utils/` and `supabase-mocks/` for shared Supabase/client helpers. Do not
put feature-only `make*` fixtures in generic `api/src/test-utils/` — see
[vitest-colocate-make-fixtures](./vitest-colocate-make-fixtures.md).

## See also

- Vitest mocking (authoring) index