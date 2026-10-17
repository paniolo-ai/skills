---
source-slug: playwright-mock-auth-in-tests
source-hash: 13cac46dbeed9ad43bf82963967fe820b8a934853063fa853406345d82be1301
bundled: 2026-10-16
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
