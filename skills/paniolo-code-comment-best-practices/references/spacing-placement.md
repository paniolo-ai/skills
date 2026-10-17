---
source-slug: spacing-placement
source-hash: 23be77ae2224cf4606876edda6f040ae13df81932476d707a7dbb09a201d3366
bundled: 2026-10-17
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
