---
source-slug: typescript-effect-refactoring-promise-to-effect
source-hash: 5148cb24093d366e09f15b9d9db4dbc96e6561258e0cdcb481021ac921fbeb23
bundled: 2026-10-16
title: Refactoring Promise-Returning Functions to Effect
type: concept
tags:
- authoring
- effect-ts
- effect
- typescript
updated: 2026-06-18
---

# Refactoring Promise-Returning Functions to Effect

When converting existing `async function(): Promise<T>` service functions to Effect, follow this structured approach.

**Identify scope:** Start with service functions (not HTTP handlers, which use `handleHttpEndpoint` directly). Prioritize functions with complex async boundaries or multiple error paths.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
