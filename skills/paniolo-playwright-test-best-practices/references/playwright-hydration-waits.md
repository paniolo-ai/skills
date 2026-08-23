---
source-slug: playwright-hydration-waits
source-hash: 0135a3181aac7240f40d5aa3dbb0b743581d5e1b7a426b88c3681fedb6efed04
bundled: 2026-08-20
title: Hydration Waits
type: concept
tags:
- authoring
- playwright
- testing
- e2e
updated: 2026-08-20
---

# Hydration Waits

Default to web-first assertions and locator retries. They are faster to read and more resilient than
sleeps.

The one documented exception in this repo is a short post-navigation hydration settle using a named
`HYDRATION_WAIT_MS` constant. Use it only when the page is known to need React hydration time after
`page.goto()` and a plain assertion is not enough.

```typescript
const HYDRATION_WAIT_MS = 2000;

// Arrange
await authenticateTestUser(page);
await page.goto("/en/dashboard");
await page.waitForTimeout(HYDRATION_WAIT_MS);

// Assert
await expect(page.getByTestId("dashboard-root")).toBeVisible();
```

Guidelines:

- ✅ Keep the wait in a named constant such as `HYDRATION_WAIT_MS`
- ✅ Use it immediately after navigation, not as a generic "fix flake" tool
- ✅ Prefer assertions and locator waiting for everything after the initial hydrate
- ❌ Add new magic-number sleeps inline
- ❌ Scatter `waitForTimeout` calls through interaction steps
- ❌ Replace ordinary web-first assertions with sleeps

## See also

- Playwright e2e testing (authoring) index
