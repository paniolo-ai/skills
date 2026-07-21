---
source-wiki: sharp-shooter-wiki
source-slug: typescript-effect-refactoring-promise-to-effect
source-hash: c1d1d2699a480eedaa619612978c6aabf90c3985bb3cab8e40f686b1449cdf76
bundled: 2026-07-20
title: Refactoring Promise-Returning Functions to Effect
type: concept
tags:
- authoring
- effect-ts
- effect
- typescript
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Refactoring Promise-Returning Functions to Effect

When converting existing `async function(): Promise<T>` service functions to Effect, follow this
structured approach.

**Identify scope:** Start with service functions (not HTTP handlers, which use `handleHttpEndpoint`
directly). Prioritize functions with complex async boundaries or multiple error paths.

## See also

- Effect-TS (authoring) index