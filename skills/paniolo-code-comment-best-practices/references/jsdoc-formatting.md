---
source-wiki: sharp-shooter-wiki
source-slug: jsdoc-formatting
source-hash: 9ae1447f5921ae5185e935489f7871dd8c3dad54ff529df0a7da73da91d6ccf1
bundled: 2026-07-20
title: JSDoc — formatting rules
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
sources:
- raw/harness-eng/songshare-ai-system/skill-code-comment-best-practices.md
---

# JSDoc — formatting rules

**No types in JSDoc for `.ts`/`.tsx` files.** TypeScript provides the types. Use plain descriptions
only.

```ts
// ❌
/** @param {string} name - The user name */

// ✅
/** @param name - The user name */

// ❌
/** @returns {Promise<string>} The entity id */

// ✅
/** @returns Resolved entity id */
```

**Don't repeat the symbol name** as the first sentence of a JSDoc. The symbol is already declared —
begin with a concise description of purpose and behavior.

**Multi-line JSDoc must start with `/**`on its own line** and end with`\*/` on its own line:

```ts
// ❌
/** Renders the inner TD content for the full-width delete confirmation UI.
 * Maintainers: This avoids alignment shifts seen in separate-row implementations.
 */

// ✅
/**
 * Renders the inner TD content for the full-width delete confirmation UI.
 * Maintainers: This avoids alignment shifts seen in separate-row implementations.
 */
```

**Single-line JSDoc** is fine for short descriptions that fit on one line:

```ts
/** Minimum allowed slide index (keeps bounds explicit and avoids magic numbers) */
const MIN*SLIDE*INDEX = 0;
```

**Max 100 characters per line.** Use multi-line JSDoc when a description exceeds that.

## See also

- Code comments (authoring) index