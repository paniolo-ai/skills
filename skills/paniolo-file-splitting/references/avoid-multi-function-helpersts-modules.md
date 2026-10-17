---
source-slug: avoid-multi-function-helpersts-modules
source-hash: bb6ef2df05f3744a3e6209f07fc7ed114cf1445cbff03df1351c62a5da2293c4
bundled: 2026-10-17
title: Avoid multi-function `*Helpers.ts` modules
type: concept
tags:
- authoring
- file-organization
updated: 2026-06-18
---

# Avoid multi-function `*Helpers.ts` modules

> **Hub:** [File organization (authoring)](./file-organization.md)

Do **not** create application modules such as `activePublicSongSubscriptionHelpers.ts` that export several named functions. That pattern is legacy; new work should use **one file per function**:

```text
src/song/active-songs/
├── mergeActivePublicSongIds.ts
├── mergeActivePublicSongIds.test.ts
├── ensureActivePublicSongSubscription.ts
└── ensureActivePublicSongSubscription.test.ts
```

If you touch an existing multi-export helper file, prefer splitting it (see
`file-splitting`) over adding another export.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
