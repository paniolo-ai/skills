---
source-wiki: sharp-shooter-wiki
source-slug: vitest-supabase-postgrest-mocking
source-hash: 97a4b85731fb779f181b5724fd15a3dea1cc0b8d181d9fa4943e4c4883ffd797
bundled: 2026-07-20
title: Supabase / Postgrest Mocking
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

# Supabase / Postgrest Mocking

Centralize unsafe casts in `asPostgrestResponse(value)` under `react/src/lib/test-utils`. Never
hand-craft the `{ data, error, count, status, statusText }` shape inline.

Helper:

- Path: `react/src/lib/test-utils/asPostgrestResponse.ts`
- Purpose: construct a minimal `PostgrestResponse<T>` for test mocks and contain the localized
  lint / `oxlint` disables

Usage:

```ts
import callSelect from "@/react/lib/supabase/client/safe-query/callSelect";
import asPostgrestResponse from "@/react/lib/test-utils/asPostgrestResponse";

vi.mock("@/react/lib/supabase/client/safe-query/callSelect");
const mockedCallSelect = vi.mocked(callSelect);

mockedCallSelect.mockResolvedValue(asPostgrestResponse({ data: [{ id: "r1" }] }));

mockedCallSelect.mockResolvedValue(
	asPostgrestResponse({ data: [], error: null, count: null, status: 200, statusText: "OK" }),
);
```

Guidelines:

- Prefer `vi.mocked(callSelect)` rather than casting to `any`.
- Avoid sprinkling `// oxlint-disable` around test files; keep any necessary disables centralized
  inside `asPostgrestResponse`.
- If you need a narrow unsafe cast in a test, move it into a test util instead of repeating it
  inline.
- For complex mock factories, provide a typed helper that returns the mocked function so test files
  stay lint-clean.
- When mocking external services like Supabase, ensure the mock structure matches the real library's
  response format, especially for error cases.

```typescript
return Promise.resolve({ data: null, error: { message: "Database failure" } });
```

## See also

- Vitest mocking (authoring) index