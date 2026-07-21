---
source-wiki: sharp-shooter-wiki
source-slug: constants-placement-across-modules
source-hash: 45e58841b7b17b46bd86dc6dc5634436d69aa6d2270399580e2c99489aaa6875
bundled: 2026-07-20
title: Constants placement across modules
type: concept
tags:
- authoring
- file-organization
updated: 2026-06-18
sources:
- raw/harness-eng/songshare-ai-system/skill-file-organization.md
---

# Constants placement across modules

> **Hub:** File organization (authoring) index

- **1 file:** module-level `const` in that file (no `constants.ts`)
- **2 files:** ask owner — colocate if tightly coupled, otherwise create shared module
- **3+ files:** create `constants.ts` or `feature-constants.ts` (name descriptively)

Rationale: keeps API surface minimal, reduces unnecessary indirection.

## See also

- File organization (authoring) index

## See also

- File organization (authoring) index