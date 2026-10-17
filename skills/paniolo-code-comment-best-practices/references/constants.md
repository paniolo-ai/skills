---
source-slug: constants
source-hash: 2ed345cb3ce6566acd88edb58bfbabe0a519de66fb3f072e139f4bf06dab57ac
bundled: 2026-10-17
title: Constants and grouped symbols
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
---

# Constants and grouped symbols

Do **not** use JSDoc to comment on more than one symbol at a time. For groups of related constants, use a single `//` above the group — not a JSDoc that spans all of them:

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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
