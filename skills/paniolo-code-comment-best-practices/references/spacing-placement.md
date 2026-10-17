---
source-slug: spacing-placement
source-hash: 1c9c75278d1ddd9953dfdd4804df4d266a75dbee12f6be15755f72575b34ef3e
bundled: 2026-10-16
title: Spacing and placement
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
---

# Spacing and placement

**One blank line above a JSDoc block.** Leave exactly one empty line before a JSDoc block to visually separate it from preceding code (unless the JSDoc is at the top of the file or immediately after an opening `{`).

```ts
// ❌ — two blank lines
const x = 1;

/**
 * Description...
 */
function f() {}

// ✅ — one blank line
const x = 1;

/**
 * Description...
 */
function f() {}
```

**No blank lines between JSDoc and its symbol.** The JSDoc must attach directly to what it
documents.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
