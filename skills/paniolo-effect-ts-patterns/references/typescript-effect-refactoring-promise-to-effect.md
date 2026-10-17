---
source-slug: typescript-effect-refactoring-promise-to-effect
source-hash: d808986981454317ba20e0bee07547b01cd7c235f384b9fd079a10d3ed00880f
bundled: 2026-10-17
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
