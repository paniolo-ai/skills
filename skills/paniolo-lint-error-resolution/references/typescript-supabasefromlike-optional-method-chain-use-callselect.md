---
source-wiki: sharp-shooter-wiki
source-slug: typescript-supabasefromlike-optional-method-chain-use-callselect
source-hash: 05da664928a3be257fb24976e98066e8ef2369e1a839057cebec393b221d8c2e
bundled: 2026-07-20
title: '`SupabaseFromLike` optional chains — use `callSelect`'
type: concept
tags:
- authoring
- typescript
- lint
- songshare
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# `SupabaseFromLike` optional chains — use `callSelect`

When oxlint flags unsafe optional chaining on `client.from(...).select(...)`, route reads through
`callSelect` (typed wrapper in `react/src/lib/supabase/client/safe-query/callSelect.ts`) instead
of open-ended `SupabaseFromLike` chains.

## See also

- TypeScript lint (authoring) index