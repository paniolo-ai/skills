---
source-slug: playwright-test
source-hash: d4e31f191683964bab461ecc622826fa7f574d6e5b3f7a9acb11ef9c89dd4dc6
bundled: 2026-10-16
title: Authoring — Playwright e2e testing
type: index
tags:
- index
- authoring
- playwright-test
updated: 2026-09-30
---

# Playwright e2e testing (authoring)

Operational reference for playwright e2e testing — loaded from skills and agents.
## Reference

### Clicks silently ignored on backgrounded pages

Symptoms: after navigating a second page (`viewerPage`, `recipientPage`), a click on the first page (`presenterPage`, `senderPage`) has no effect in WebKit but works in Chromium.

Cause: WebKit does not dispatch input events to pages that are not in the foreground. Call `page.bringToFront()` before clicking whenever you have interleaved multi-page navigation:

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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
