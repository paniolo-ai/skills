---
source-wiki: sharp-shooter-wiki
source-slug: typescript-optional-method-chain-use-typed-query-wrapper
source-hash: cf3fe0108970e98833e04d82aefd4ab1b0764b31e6e855df8298aa260037ec4a
bundled: 2026-08-01
title: Query-builder optional chains — use a typed wrapper
type: concept
tags:
- authoring
- typescript
- lint
updated: 2026-07-24
---

# Query-builder optional chains — use a typed wrapper

When oxlint flags unsafe optional chaining on a fluent query builder such as
`client.from(...).select(...)`, route reads through a typed wrapper (for example a
`callSelect` helper in your data-access layer) instead of open-ended optional chains on
the builder's own interface.

The wrapper gives the call site one concrete return type, so the optional chain
disappears rather than being suppressed.

## See also

- TypeScript lint (authoring) index