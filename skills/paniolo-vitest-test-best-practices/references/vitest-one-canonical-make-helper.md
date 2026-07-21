---
source-wiki: sharp-shooter-wiki
source-slug: vitest-one-canonical-make-helper
source-hash: 3d0c54dde875acc8edf077c4f5dca9c5ba0e646df32b6cf0326e37293a9f0731
bundled: 2026-07-20
title: One canonical `make*` per fixture type
type: concept
tags:
- authoring
- vitest
- testing
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# One canonical `make*` per fixture type

Each fixture shape (e.g. `EventParticipant`, `PlaylistLibraryEntry`, `playlist_library` row)
should have **one** default-exported builder in a `*.test-util.ts` file. All tests import that
helper; they do not reimplement the same field list in another file.

**Before adding** `makeFoo.test-util.ts`, search the repo:

```bash
rg "makeFoo|function make.*Participant|function make.*LoadedEvent" --glob "*.test-util.ts" --glob "*test*"
npm run qmd -- search "make EventParticipant test fixture"
```

| Do                                                                          | Do not                                                                                    |
| --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `import makeEventParticipant from ".../makeEventParticipant.test-util"`     | A second `function makeParticipant()` in another spec with the same defaults              |
| `makeManageViewLoadedEvent({ event_name: EVENT_NAME, ...overrides })`       | Copy/paste the same `makeEventPublic({ ... })` block in three test files                  |
| One file-level `const PRESET = makeCanonical({ ...this file's constants })` | Two different `*.test-util.ts` files that both build `EventParticipant`                   |
| Extend the canonical helper when a new field is needed everywhere           | A thin wrapper that hides a duplicate default list — wrappers must delegate, not redefine |

**Allowed in a single test file:**

- `Partial` overrides at the call site:
  `makeEventParticipant({ user_id: ADMIN_ID, role: "event_admin" })`.
- One preset constant built from the canonical helper (e.g. `const MANAGE_VIEW_LOADED_EVENT =
makeManageViewLoadedEvent({ ... })`).
- One preset function in that file that **only** spreads file constants into the canonical helper
  (e.g. `makeHookLoadedEvent` in `useEventManageView.test.tsx` calling `makeManageViewLoadedEvent`).

**Not allowed:** the same fixture type with two or more independent default implementations in
different directories (merge into one canonical `*.test-util.ts` and update imports).

## See also

- Vitest unit testing (authoring) index