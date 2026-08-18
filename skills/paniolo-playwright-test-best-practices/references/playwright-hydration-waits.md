---
source-wiki: sharp-shooter-wiki
source-slug: playwright-hydration-waits
source-hash: 0c8ed2620acc2b5e8cce086cdbf14a48dacfe30baee9bd95a3eac5d1ec2add40
bundled: 2026-08-17
title: Hydration Waits
type: concept
tags:
- authoring
- playwright
- testing
- e2e
updated: 2026-06-18
---

# Hydration Waits

Default to web-first assertions and locator retries. They are faster to read and more resilient than
sleeps.

The one documented exception in this repo is a short post-navigation hydration settle using a named
`HYDRATION*WAIT*MS` constant. Use it only when the page is known to need React hydration time after
`page.goto()` and a plain assertion is not enough.

```typescript
const HYDRATION*WAIT*MS = 2000;

// Arrange
await authenticateTestUser(page);
await page.goto("/en/dashboard");
await page.waitForTimeout(HYDRATION*WAIT*MS);

// Assert
await expect(page.getByTestId("dashboard-root")).toBeVisible();
```

Guidelines:

- ✅ Keep the wait in a named constant such as `HYDRATION*WAIT*MS`
- ✅ Use it immediately after navigation, not as a generic "fix flake" tool
- ✅ Prefer assertions and locator waiting for everything after the initial hydrate
- ❌ Add new magic-number sleeps inline
- ❌ Scatter `waitForTimeout` calls through interaction steps
- ❌ Replace ordinary web-first assertions with sleeps

## See also

- Playwright e2e testing (authoring) index
