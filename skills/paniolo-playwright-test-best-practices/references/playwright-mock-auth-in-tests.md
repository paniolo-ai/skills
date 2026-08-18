---
source-wiki: sharp-shooter-wiki
source-slug: playwright-mock-auth-in-tests
source-hash: 5131388b24a4c57f02b645c189824cdc8fa24c85a6fc898732bcca0c2919e4b6
bundled: 2026-08-17
title: Mock Auth in Tests
type: concept
tags:
- authoring
- playwright
- testing
- e2e
- auth
updated: 2026-06-18
---

# Mock Auth in Tests

The app uses OAuth with HttpOnly cookies. Instead of going through full OAuth flows in tests (slow,
requires test accounts), lighter tests mock the `/api/me` endpoint using `page.route()`.

## See also

- Playwright auth testing (authoring) index
