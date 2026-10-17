---
source-slug: typescript-optional-method-chain-use-typed-query-wrapper
source-hash: f157cf979e40f748226bdfe97dd7dfe9372b0853ead64313b6d500c4554d1137
bundled: 2026-10-16
title: Query-builder optional chains — use a typed wrapper
type: concept
tags:
- authoring
- typescript
- lint
updated: 2026-07-24
---

# Query-builder optional chains — use a typed wrapper

When oxlint flags unsafe optional chaining on a fluent query builder such as `client.from(...).select(...)`, route reads through a typed wrapper (for example a `callSelect` helper in your data-access layer) instead of open-ended optional chains on the builder's own interface.

The wrapper gives the call site one concrete return type, so the optional chain disappears rather than being suppressed.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
