---
source-wiki: sharp-shooter-wiki
source-slug: spacing-placement
source-hash: 76b222324e91cbde9983188559d62816d720898d4ac50bdd3386a47750671de0
bundled: 2026-07-20
title: Spacing and placement
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
sources:
- raw/harness-eng/songshare-ai-system/skill-code-comment-best-practices.md
---

# Spacing and placement

**One blank line above a JSDoc block.** Leave exactly one empty line before a JSDoc block to
visually separate it from preceding code (unless the JSDoc is at the top of the file or immediately
after an opening `{`).

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

## See also

- Code comments (authoring) index