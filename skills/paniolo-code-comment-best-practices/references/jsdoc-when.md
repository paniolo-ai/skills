---
source-slug: jsdoc-when
source-hash: 69a355c6caa04535f3e2434c953eec749b19727f0c4892d65c20007c30c70837
bundled: 2026-10-16
title: JSDoc — when to use
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
---

# JSDoc — when to use

Use `/** */` above:

- Exported functions and components
- Exported types and interfaces
- Non-obvious constants (single-line `/** description */` is fine)

- Every exported function and any non-trivial internal function (complex logic, side effects, or multiple branches) should include JSDoc describing purpose, side-effects, and usage.
- In tests and helper-heavy files, named local helpers and local guards should also get JSDoc when they are reused, injected, or do more than a trivial one-liner. Common examples are `installMockBody`, `makeProps`, `LocationProbe`, `noop`, `noopAsync`, `noopUnsubscribe`, and type guards added to satisfy strict typing.

Do **not** use JSDoc:

- Above `describe` or `it`/`test` blocks in test files (use `//` instead — see [§6](./test-comments.md))
- To comment on more than one symbol at a time (see [§7](./constants.md))
- When a single-line `//` above a non-exported helper is sufficient

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
