---
source-wiki: sharp-shooter-wiki
source-slug: playwright-test
source-hash: 6f0c20d5e1367b61dddef5aa306a4b0d2bba2e5e66d460d94eb1711dd5626cc9
bundled: 2026-07-20
title: Authoring — Playwright e2e testing
type: index
tags:
- index
- authoring
- playwright-test
updated: 2026-06-18
---

# Playwright e2e testing (authoring)

Operational reference for playwright e2e testing — loaded from skills and agents.
Repo hub: `docs/testing/playwright-test-best-practices.md` (thin router during wiki migration).

## Pages

- AAA Pattern
- Best practices
- Auth/session helpers
- Authentication
- Base URL Rules
- Browser Projects
- CI Browser Caching
- Config
- Debugging
- Deployed environment runs
- Edge cases
- Environment
- Environment Variables
- Everyday local runs
- High-Level Model
- Hydration Waits
- Lighthouse commands
- Lighthouse Environment Variables
- Lighthouse Recommendations
- Lighthouse Troubleshooting
- Local runner wrappers
- Logs and Temp Files
- macOS Notes
- Navigation patterns
- Realtime timing failures
- Routes that stay as pages
- Running the example
- Session/auth failures
- Staging DB runs
- Startup failures
- Tab URLs (canonical)
- Test Organization
- Test-Owned Entities
- Translation-Aware Tests
- VS Code Integration
- What Happens During a Local Run

## Reference

### Clicks silently ignored on backgrounded pages

Symptoms: after navigating a second page (`viewerPage`, `recipientPage`), a click on the first page
(`presenterPage`, `senderPage`) has no effect in WebKit but works in Chromium.

Cause: WebKit does not dispatch input events to pages that are not in the foreground.
Call `page.bringToFront()` before clicking whenever you have interleaved multi-page navigation:

```typescript
// Arrange: navigate the viewer
await viewerPage.goto(`${BASE_URL}/en/event/${slug}/slideshow`, { waitUntil: "load" });

// Act: switch focus back to the presenter page before clicking
await presenterPage.bringToFront();
await presenterPage.getByRole("button", { name: /next/i }).first().click();
```

### Missing `ignoreHTTPSErrors`

Symptoms: API calls return 500 or fail silently in WebKit only.
Actions (slide nav, image upload, and similar) fire but never resolve.
Assertions commonly wait out the full timeout.

Cause: WebKit rejects self-signed TLS certificates.

Every project in `playwright.config.ts` must set `ignoreHTTPSErrors: true`.
This is required for the `webkit` project as well.

The top-level `use` block does not automatically cascade into project-level
`use` blocks, so the option must be repeated per project.

## See also

- Authoring index
- Repo hub: `docs/testing/playwright-test-best-practices.md`