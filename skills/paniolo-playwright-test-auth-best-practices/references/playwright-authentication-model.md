---
source-wiki: sharp-shooter-wiki
source-slug: playwright-authentication-model
source-hash: 59192cbcb959cf56c6db39ba1752f21dd7a4da675e8fc9a34330c620dbfe010b
bundled: 2026-07-20
title: Authentication Model
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

# Authentication Model

Two main auth strategies:

1. **Mocked/local auth helpers** — for lighter tests that do not need real staging users; intercepts
   `/api/me` with `page.route()` — see [Mock Auth in Tests](./playwright-mock-auth-in-tests.md)
2. **Real signed session cookies** — for staging-db and deployed staging runs; session files live in
   `e2e/.auth/google-user.json` and `e2e/.auth/google-user-2.json` — see [Staging DB
   Setup](./playwright-staging-db-setup.md)

The real-session flow does not drive OAuth in the browser. The helper script signs a `userSession`
cookie and stores it as Playwright `storageState`.

## See also

- Playwright auth testing (authoring) index