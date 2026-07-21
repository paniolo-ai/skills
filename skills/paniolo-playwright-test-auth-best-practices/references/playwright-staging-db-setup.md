---
source-wiki: sharp-shooter-wiki
source-slug: playwright-staging-db-setup
source-hash: 15faf988dbecd7d474260356d6195160596fbdb9c7c1caeaa4602a8f01ea839f
bundled: 2026-07-20
title: Staging DB Setup
type: concept
tags:
- authoring
- playwright
- testing
- e2e
- auth
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Staging DB Setup

Use real signed sessions (instead of mocked `/api/me`) when tests need Realtime subscriptions,
actual DB rows, or RLS enforcement.

| Mode                        | Frontend           | API                    | Supabase |
| --------------------------- | ------------------ | ---------------------- | -------- |
| **Local site + staging DB** | `localhost:5173`   | `localhost:8787`       | staging  |
| **Staging site**            | `<staging-domain>` | `<staging-domain>/api` | staging  |

## See also

- Playwright auth testing (authoring) index