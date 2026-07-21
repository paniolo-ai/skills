---
source-wiki: sharp-shooter-wiki
source-slug: playwright-mock-auth-best-practices
source-hash: e01636bbd02af7a5079211e9f62ed7cd55565f0285da4db8246e6705efd66f8d
bundled: 2026-07-20
title: Mock Auth Best Practices
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

# Mock Auth Best Practices

- **Always call authentication before navigation** — calling it after may cause race conditions
- **Use separate contexts for multiple users** — shared contexts cause conflicts
- **Use descriptive user data** — `createTestUser({ name: "Admin User" })` not `createTestUser({
name: "User" })`
- **Test both authenticated and unauthenticated states** for any protected route

```typescript
test.describe("Dashboard Access", () => {
	test("authenticated user can access", async ({ page }) => {
		await authenticateTestUser(page);
		// ... test authenticated behavior
	});

	test("unauthenticated user is redirected", async ({ page }) => {
		await mockSignedOutUser(page);
		// ... test redirect behavior
	});
});
```

## See also

- Playwright auth testing (authoring) index