---
source-slug: jsdoc-formatting
source-hash: e699e4451caf8707a2bde79e9fd444604dda99885ebcb8740ecd2158aa87784a
bundled: 2026-10-17
title: JSDoc — formatting rules
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
---

# JSDoc — formatting rules

**No types in JSDoc for `.ts`/`.tsx` files.** TypeScript provides the types. Use plain descriptions only.

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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
