---
source-wiki: sharp-shooter-wiki
source-slug: playwright-multiple-users-mock
source-hash: 425af29ae9b0be8dcc1e7e03222bc6fd7ecb751cbeb9d2bec9d2afc11f1c6069
bundled: 2026-07-20
title: Multiple Users (Mock)
type: concept
tags:
- authoring
- playwright
- testing
- e2e
- auth
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Multiple Users (Mock)

Use separate browser contexts — never share a context between users:

```typescript
test("different users see their own data", async ({ browser }) => {
	const context1 = await browser.newContext();
	const context2 = await browser.newContext();
	try {
		const page1 = await context1.newPage();
		const page2 = await context2.newPage();
		await authenticateTestUser(page1, createTestUser({ name: "User One" }));
		await authenticateTestUser(page2, createTestUser({ name: "User Two" }));
		await page1.goto("/en/dashboard");
		await page2.goto("/en/dashboard");
		await expect(page1.getByText(/user one/i)).toBeVisible();
		await expect(page2.getByText(/user two/i)).toBeVisible();
	} finally {
		await context1.close();
		await context2.close();
	}
});
```

## See also

- Playwright auth testing (authoring) index