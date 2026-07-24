---
source-wiki: sharp-shooter-wiki
source-slug: test-comments
source-hash: 12ab309d1af16c04d21bf8f3a5c01be91f16ad7c2a23b8dcde8fc907391dcc68
bundled: 2026-07-24
title: Test file comments
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
---

# Test file comments

Use `//` in test files to describe purpose or complex setup. Do **not** use JSDoc above `describe`
or `it`/`test` blocks. Only add a comment above a test block if the test name is not
self-explanatory. Keep these comments extremely concise.

```ts
// ❌ — redundant header + over-explained
// Test suite for `subscribeToActiveItems`.
//
// Ensures the factory returns a no-op unsubscribe, warns correctly when there
// is no database client or no active IDs, and logs errors when token fetching fails.
describe("subscribeToActiveItems", () => {

// ✅
// Ensures the factory returns a no-op unsubscribe, warns correctly when there
// is no database client or no active IDs, and logs errors when token fetching fails.
describe("subscribeToActiveItems", () => {
```

Use JSDoc in test files only for shared utility functions that are exported or reused across test
files.

## See also

- Code comments (authoring) index