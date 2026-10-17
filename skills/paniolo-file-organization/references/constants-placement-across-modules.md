---
source-slug: constants-placement-across-modules
source-hash: a9cb494ff01daed7ceff34ce7609171e14235823c8c3ee85d4779f4d2bd94eef
bundled: 2026-10-17
title: Constants placement across modules
type: concept
tags:
- authoring
- file-organization
updated: 2026-06-18
---

# Constants placement across modules

> **Hub:** [File organization (authoring)](./file-organization.md)

- **1 file:** module-level `const` in that file (no `constants.ts`)
- **2 files:** ask owner — colocate if tightly coupled, otherwise create shared module
- **3+ files:** create `constants.ts` or `feature-constants.ts` (name descriptively)

Rationale: keeps API surface minimal, reduces unnecessary indirection.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
