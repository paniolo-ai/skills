---
source-slug: playwright-mock-auth-troubleshooting
source-hash: 965e1008877b9c716be7fc98a2ae50a08d9217368689e602ab7f3908b518235f
bundled: 2026-08-20
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

**User data not showing up** — use web-first assertions first. If the page has a known
post-navigation hydration delay, use the documented `HYDRATION_WAIT_MS` pattern instead of an
arbitrary sleep:

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

## See also

- Playwright auth testing (authoring) index
