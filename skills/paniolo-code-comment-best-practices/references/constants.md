---
source-wiki: sharp-shooter-wiki
source-slug: constants
source-hash: c3396f73668a24980f3937aaa451b9c2c178a895f7ec877fbf79b43270ba9535
bundled: 2026-07-20
title: Constants and grouped symbols
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
sources:
- raw/harness-eng/songshare-ai-system/skill-code-comment-best-practices.md
---

# Constants and grouped symbols

Do **not** use JSDoc to comment on more than one symbol at a time. For groups of related constants,
use a single `//` above the group — not a JSDoc that spans all of them:

```ts
// ❌
/** Numeric constants used in assertions to make expected values explicit. */
const ZERO = 0;
const ONE = 1;
const TWO = 2;

// ✅
// Numeric constants used in assertions to make expected values explicit.
const ZERO = 0;
const ONE = 1;
const TWO = 2;
```

For a single non-obvious constant, prefer single-line JSDoc:

```ts
/** Minimum allowed slide index (keeps bounds explicit and avoids magic numbers) */
const MIN*SLIDE*INDEX = 0;
```

## See also

- Code comments (authoring) index