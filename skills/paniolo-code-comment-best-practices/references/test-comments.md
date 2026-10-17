---
source-slug: test-comments
source-hash: dcf7f76d28cd64c10036b5bf4f0aba3faae2fd5fc476a008bca3bada9e86e0f9
bundled: 2026-10-17
title: Test file comments
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
---

# Test file comments

Use `//` in test files to describe purpose or complex setup. Do **not** use JSDoc above `describe` or `it`/`test` blocks. Only add a comment above a test block if the test name is not self-explanatory. Keep these comments extremely concise.

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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
