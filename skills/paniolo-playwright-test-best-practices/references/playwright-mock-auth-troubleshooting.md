---
source-wiki: sharp-shooter-wiki
source-slug: playwright-mock-auth-troubleshooting
source-hash: 1a2ef80fe825587bbe4f902baa727ab9f5aa80404c737b38c80150207d0e6096
bundled: 2026-08-17
title: Mock Auth Troubleshooting
type: concept
tags:
- authoring
- playwright
- testing
- e2e
- auth
updated: 2026-06-18
---

# Mock Auth Troubleshooting

**Test fails with "Not authenticated"** — call `authenticateTestUser(page)` before `page.goto()`.

**User data not showing up** — use web-first assertions first. If the page has a known
post-navigation hydration delay, use the documented `HYDRATION*WAIT*MS` pattern instead of an
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
