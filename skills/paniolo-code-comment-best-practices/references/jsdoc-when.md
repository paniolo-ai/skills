---
source-wiki: sharp-shooter-wiki
source-slug: jsdoc-when
source-hash: 0d76639986f769e96d0500a468e400e82f2f2abf719c0e498d5a36783e8a903b
bundled: 2026-07-20
title: JSDoc — when to use
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
sources:
- raw/harness-eng/songshare-ai-system/skill-code-comment-best-practices.md
---

# JSDoc — when to use

Use `/** */` above:

- Exported functions and components
- Exported types and interfaces
- Non-obvious constants (single-line `/** description */` is fine)

- Every exported function and any non-trivial internal function (complex logic, side effects, or
  multiple branches) should include JSDoc describing purpose, side-effects, and usage.
- In tests and helper-heavy files, named local helpers and local guards should also get JSDoc when
  they are reused, injected, or do more than a trivial one-liner. Common examples in this repo are
  `installMockBody`, `makeProps`, `LocationProbe`, `noop`, `noopAsync`, `noopUnsubscribe`, and
  type guards added to satisfy strict typing.

Do **not** use JSDoc:

- Above `describe` or `it`/`test` blocks in test files (use `//` instead — see [§6](./test-comments.md))
- To comment on more than one symbol at a time (see [§7](./constants.md))
- When a single-line `//` above a non-exported helper is sufficient

## See also

- Code comments (authoring) index