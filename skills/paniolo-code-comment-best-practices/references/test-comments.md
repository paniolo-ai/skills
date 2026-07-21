---
source-wiki: sharp-shooter-wiki
source-slug: test-comments
source-hash: 2067a98b6a54c3d528619c46c0ee0365603eeeb8685c6cdcb9baf49300a222b8
bundled: 2026-07-20
title: Test file comments
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
sources:
- raw/harness-eng/songshare-ai-system/skill-code-comment-best-practices.md
---

# Test file comments

Use `//` in test files to describe purpose or complex setup. Do **not** use JSDoc above `describe`
or `it`/`test` blocks. Only add a comment above a test block if the test name is not
self-explanatory. Keep these comments extremely concise.

```ts
// ❌ — redundant header + over-explained
// Test suite for `subscribeToActivePrivateSongs`.
//
// Ensures the factory returns a no-op unsubscribe, warns correctly when there
// is no Supabase client or no active IDs, and logs errors when token fetching fails.
describe("subscribeToActivePrivateSongs", () => {

// ✅
// Ensures the factory returns a no-op unsubscribe, warns correctly when there
// is no Supabase client or no active IDs, and logs errors when token fetching fails.
describe("subscribeToActivePrivateSongs", () => {
```

Use JSDoc in test files only for shared utility functions that are exported or reused across test
files.

## See also

- Code comments (authoring) index