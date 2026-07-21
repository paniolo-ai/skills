---
source-wiki: sharp-shooter-wiki
source-slug: playwright-aaa-pattern
source-hash: e3c9c9e513c5e4543f38022a98b84ba4a42c200838e23667f7b8a990094bc14a
bundled: 2026-07-20
title: AAA Pattern
type: concept
tags:
- authoring
- playwright
- testing
- e2e
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# AAA Pattern

Structure every test with three clearly commented phases — **Arrange**, **Act**, **Assert**:

```typescript
test("user can add a song to their library", async ({ page }) => {
	// Arrange
	await authenticateTestUser(page);
	await page.goto("/en/song-library");
	await expect(page.getByRole("heading", { name: /song library/i })).toBeVisible();

	// Act
	await page.getByRole("button", { name: /add song/i }).click();
	await page.getByLabel("Song title").fill("My New Song");
	await page.getByRole("button", { name: /save/i }).click();

	// Assert
	await expect(page.getByText("My New Song")).toBeVisible();
});
```

**Why it matters:**

- Makes test intent clear at a glance — you can tell what state is being set up, what the user does,
  and what the expected outcome is
- Makes failures easier to diagnose — if the Act step throws, the Arrange wasn't the problem
- Discourages mixing assertions into the middle of interactions

**For multi-step flows**, use inline comments to separate phases within a longer test rather than
collapsing everything into a single block:

```typescript
test("sender shares a song and recipient accepts", async ({ browser }) => {
	// Arrange — set up two authenticated contexts
	const senderCtx = await browser.newContext({ storageState: GOOGLE * USER * SESSION_PATH });
	const recipientCtx = await browser.newContext({
		storageState: GOOGLE * USER * SESSION * PATH * 2,
	});
	const senderPage = await senderCtx.newPage();
	const recipientPage = await recipientCtx.newPage();

	try {
		// Act — sender shares
		await senderPage.goto("/en/song/my-test-song");
		await senderPage.getByRole("button", { name: /share/i }).click();
		await senderPage.getByLabel("Username").fill("user2");
		await senderPage.getByRole("option", { name: "user2" }).click();
		await senderPage.getByRole("button", { name: /send/i }).click();

		// Assert — recipient sees the invitation
		await recipientPage.goto("/en/notifications");
		await expect(recipientPage.getByText(/shared a song with you/i)).toBeVisible();

		// Act — recipient accepts
		await recipientPage.getByRole("button", { name: /accept/i }).click();

		// Assert — song appears in recipient's library
		await recipientPage.goto("/en/song-library");
		await expect(recipientPage.getByText("my-test-song")).toBeVisible();
	} finally {
		await senderCtx.close();
		await recipientCtx.close();
	}
});
```

## See also

- Playwright e2e testing (authoring) index