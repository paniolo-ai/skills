---
source-slug: avoid-multi-function-helpersts-modules
source-hash: 0a323f9a8e87905a8e217a3f3231883f277b0f47b1714867dd210eed62afd820
bundled: 2026-10-16
title: Avoid multi-function `*Helpers.ts` modules
type: concept
tags:
- authoring
- file-organization
updated: 2026-06-18
---

# Avoid multi-function `*Helpers.ts` modules

> **Hub:** File organization (authoring) index

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
