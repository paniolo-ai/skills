---
source-slug: vitest-colocate-make-fixtures
source-hash: 4665d7f79011f13b06923e44f2bd6c7223d239821259bdc665d4fa6f6770e7e6
bundled: 2026-10-17
title: Where to place `make*` helpers
type: concept
tags:
- authoring
- vitest
- testing
updated: 2026-07-24
---

# Where to place `make*` helpers

Colocate feature-specific `make*` files with the code that owns the type:

| Fixture kind                                 | Location                                                     | Examples                                              |
| -------------------------------------------- | ------------------------------------------------------------ | ----------------------------------------------------- |
| Feature/domain row or DTO                    | Same feature folder as the handler or type                   | `api/src/item-library/makeItemLibraryRow.test-util.ts` |
| Sub-area within a domain                     | That subfolder (`item-entry/`, `test-utils/` under feature)  | `react/src/item/item-entry/makeItemEntry.test-util.ts` |
| App-wide schema defaults                     | `shared/test-utils/`                                         | `makeItemPublic.test-util.ts`                          |
| Generic test infra (casts, router, response) | `react/src/lib/test-utils/`                                  | `forceCast`, `RouterWrapper`                           |
| Reusable data-client mocks                   | `api/src/test-utils/`, `client-mocks/`                       | `makeDbClient.test-util.ts`                            |

Rules:

- **Do not** add a feature-only `make*` helper under `api/src/test-utils/` or `react/src/lib/test-utils/` when only one feature uses it — put it beside that feature (same rule as `addItemsToUserLibrary.test-util.ts` in `api/src/item-library/`).
- **Before creating** a new helper, search the feature folder and nearby `*.test-util.ts` files; extend an existing `make*` rather than duplicating a local function in each spec.
- **Imports:** use `./makeFoo.test-util` from tests in the same folder; use package-aliased paths (`@api/...` / `@react/...`) when calling from another package area (see file-organization skill for cross-module paths).

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
