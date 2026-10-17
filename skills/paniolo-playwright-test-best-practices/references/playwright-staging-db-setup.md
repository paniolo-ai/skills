---
source-slug: playwright-staging-db-setup
source-hash: 8a697cba8bd097bd9ce72343aec12006b53b920837a375a44e03e1e9f3138b04
bundled: 2026-10-16
title: Staging DB Setup
type: concept
tags:
- authoring
- playwright
- testing
- e2e
- auth
updated: 2026-06-18
---

# Staging DB Setup

Use real signed sessions (instead of mocked `/api/me`) when tests need Realtime subscriptions, actual DB rows, or RLS enforcement.

| Mode                        | Frontend           | API                    | Supabase |
| --------------------------- | ------------------ | ---------------------- | -------- |
| **Local site + staging DB** | `localhost:5173`   | `localhost:8787`       | staging  |
| **Staging site**            | `<staging-domain>` | `<staging-domain>/api` | staging  |

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
