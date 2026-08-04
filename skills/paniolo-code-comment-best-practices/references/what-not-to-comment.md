---
source-wiki: sharp-shooter-wiki
source-slug: what-not-to-comment
source-hash: c9b50c78937dd13eb3cfcb9b8bf5ae5d7dd8cb7003b86fb70cf4f31aaad11687
bundled: 2026-08-01
title: What NOT to comment
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
---

# What NOT to comment

**Obvious code** — if the code is self-explanatory, no comment is needed:

```ts
// ❌ redundant
// Increment counter
counter++;

// ✅ no comment needed
counter++;
```

**TypeScript types** — don't repeat type information that's already in the signature:

```ts
// ❌ restates types
/** @param id - Customer ID (string) */

// ✅ describes behavior
/** @param id - Unique identifier used to look up the record */
```

**Code that should be refactored** — if logic is complex enough to need a paragraph of explanation,
consider renaming, extracting a function, or simplifying first. Only add the comment if complexity
is genuinely unavoidable due to a technical constraint. **"No hazard lights"** — don't apologize for
messy code; fix it.

## See also

- Code comments (authoring) index