---
source-slug: playwright-mock-auth-troubleshooting
source-hash: f7f01898dca8ef930cc269c4fd6773376fd0312f05d9cb018ed0b45c5f016818
bundled: 2026-10-16
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
