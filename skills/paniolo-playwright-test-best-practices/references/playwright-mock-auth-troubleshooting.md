---
source-slug: playwright-mock-auth-troubleshooting
source-hash: 078600dc57ad2b6df88433237c45154c99376d2e5db4f3ab2871e0279bafa259
bundled: 2026-10-17
title: Mock Auth Troubleshooting
type: concept
tags:
- authoring
- playwright
- testing
- e2e
- auth
updated: 2026-08-20
---

# Mock Auth Troubleshooting

**Test fails with "Not authenticated"** — call `authenticateTestUser(page)` before `page.goto()`.

**User data not showing up** — use web-first assertions first. If the page has a known post-navigation hydration delay, use the documented `HYDRATION_WAIT_MS` pattern instead of an arbitrary sleep:

```typescript
// ✅ Playwright retries until visible or times out
await expect(page.getByText(/welcome/i)).toBeVisible({ timeout: 10_000 });

// ✅ Narrow exception for known hydration settle after page.goto()
await page.waitForTimeout(HYDRATION * WAIT * MS);

// ❌ Arbitrary sleep — brittle and slow
await page.waitForTimeout(1375);
```

**Route mock not working** — verify the pattern matches: `**/api/me` matches both
`http://localhost:8787/api/me` and `https://localhost:5173/api/me`.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
