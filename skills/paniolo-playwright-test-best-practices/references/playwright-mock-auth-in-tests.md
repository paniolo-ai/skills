---
source-slug: playwright-mock-auth-in-tests
source-hash: 058e7bd4b25e734f6625b11cc27a9006215696b170ac7b6bb988601b56256572
bundled: 2026-10-17
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

The app uses OAuth with HttpOnly cookies. Instead of going through full OAuth flows in tests (slow, requires test accounts), lighter tests mock the `/api/me` endpoint using `page.route()`.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
