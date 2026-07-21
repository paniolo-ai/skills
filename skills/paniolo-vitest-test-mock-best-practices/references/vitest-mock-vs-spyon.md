---
source-wiki: sharp-shooter-wiki
source-slug: vitest-mock-vs-spyon
source-hash: 882d87fe93a688fdb6b7b900f7ed6f21ff2eadb3789392e2642424d11d2e0667
bundled: 2026-07-20
title: '`vi.mock()` vs `vi.spyOn()` - Default to `vi.mock()`'
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

# `vi.mock()` vs `vi.spyOn()` - Default to `vi.mock()`

In this repository, use `vi.mock()` as the default for dependencies imported by the SUT. This is
more predictable in ESM-heavy code and gives stronger module-boundary isolation.

```ts
// ✅ Preferred default: non-factory module mock (default export)
import clientLocalDateToUtcTimestamp from "@/shared/utils/date/clientLocalDateToUtcTimestamp";

vi.mock("@/shared/utils/date/clientLocalDateToUtcTimestamp");
vi.mocked(clientLocalDateToUtcTimestamp).mockReturnValue("2026-01-01T00:00:00Z");
```

## See also

- Vitest mocking (authoring) index