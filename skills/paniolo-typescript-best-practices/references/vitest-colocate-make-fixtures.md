---
source-wiki: sharp-shooter-wiki
source-slug: vitest-colocate-make-fixtures
source-hash: 0e0b5e14c1f5f3f8791cacf5c088451bbef117f212989ce7e40eec6b029b6209
bundled: 2026-07-20
---
---
title: "Where to place `make*` helpers"
type: concept
tags: [authoring, vitest, testing]
updated: 2026-06-18
sources:
  - raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Where to place `make*` helpers

Colocate feature-specific `make*` files with the code that owns the type:

| Fixture kind                                  | Location                                                     | Examples                                                       |
| --------------------------------------------- | ------------------------------------------------------------ | -------------------------------------------------------------- |
| Feature/domain row or DTO                     | Same feature folder as the handler or type                   | `api/src/playlist-library/makePlaylistLibraryRow.test-util.ts` |
| Sub-area within a domain                      | That subfolder (`event-entry/`, `test-utils/` under feature) | `react/src/event/event-entry/makeEventEntry.test-util.ts`      |
| App-wide schema defaults                      | `shared/test-utils/`                                         | `makeEventPublic.test-util.ts`                                 |
| Generic test infra (casts, router, PostgREST) | `react/src/lib/test-utils/`                                  | `forceCast`, `RouterWrapper`                                   |
| Reusable Supabase/client mocks                | `api/src/test-utils/`, `supabase-mocks/`                     | `makeSupabaseClient.test-util.ts`                              |

Rules:

- **Do not** add a feature-only `make*` helper under `api/src/test-utils/` or
  `react/src/lib/test-utils/` when only one feature uses it — put it beside that feature
  (same rule as `addPlaylistSongsToUserLibrary.test-util.ts` in `api/src/playlist-library/`).
- **Before creating** a new helper, search the feature folder and nearby `*.test-util.ts` files;
  extend an existing `make*` rather than duplicating a local function in each spec.
- **Imports:** use `./makeFoo.test-util` from tests in the same folder; use `@/api/...` /
  `@/react/...` when calling from another package area (see file-organization skill for
  cross-module paths).

## See also

- Vitest unit testing (authoring) index
