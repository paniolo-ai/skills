---
source-wiki: sharp-shooter-wiki
source-slug: avoid-multi-function-helpersts-modules
source-hash: 9241e15a78df46025f85db866e5bd766675b802e3bc02e92c357164efe9ccd67
bundled: 2026-07-20
title: Avoid multi-function `*Helpers.ts` modules
type: concept
tags:
- authoring
- file-organization
updated: 2026-06-18
sources:
- raw/harness-eng/songshare-ai-system/skill-file-organization.md
---

# Avoid multi-function `*Helpers.ts` modules

> **Hub:** File organization (authoring) index

Do **not** create application modules such as `activePublicSongSubscriptionHelpers.ts` that export
several named functions. That pattern is legacy; new work should use **one file per function**:

```text
src/song/active-songs/
├── mergeActivePublicSongIds.ts
├── mergeActivePublicSongIds.test.ts
├── ensureActivePublicSongSubscription.ts
└── ensureActivePublicSongSubscription.test.ts
```

If you touch an existing multi-export helper file, prefer splitting it (see
[`file-splitting`](@harness/.agents/skills/file-splitting/SKILL.md)) over adding another export.

## See also

- File organization (authoring) index

## See also

- File organization (authoring) index