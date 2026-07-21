---
source-wiki: sharp-shooter-wiki
source-slug: playwright-translation-aware-tests
source-hash: 9cbb4e995f7b7768d676441eae16e9913e0c45a44cb4e2e42d1081b87fbf5524
bundled: 2026-07-20
title: Translation-Aware Tests
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

# Translation-Aware Tests

Tests that rely on text content break when translations change. Use semantic data attributes
instead.

**Problem:**

```typescript
// ❌ Fragile — breaks when copy changes
await expect(page.getByText("Your account has been successfully deleted.")).toBeVisible();
```

**Solution — `data-testid` for element selection, semantic attributes for state:**

```typescript
// ✅ Robust — doesn't depend on text content
const alert = page.getByTestId("dismissible-alert");
await expect(alert).toBeVisible();
await expect(alert).toHaveAttribute("data-alert-type", "deleteSuccess");
await expect(alert).toHaveAttribute("data-variant", "success");
```

**Component pattern:**

```tsx
<div data-testid="dismissible-alert" data-alert-type={alertType} data-variant={variant}>
	<strong data-testid="alert-title">{title}</strong>
	<div data-testid="alert-message">{children}</div>
	<button data-testid="alert-dismiss-button" aria-label="Close">
		×
	</button>
</div>
```

**Use constants for alert types** (in `e2e/utils/translationHelpers.ts`):

```typescript
export const ALERT_TYPES = {
	DELETE_SUCCESS: "deleteSuccess",
	SIGN*OUT*SUCCESS: "signOutSuccess",
	SIGN*IN*SUCCESS: "signedInSuccess",
} as const;
```

**Guidelines:**

- ✅ Use `data-testid` for element selection
- ✅ Use semantic `data-*` attributes for state verification
- ✅ Test behavior and functionality, not specific text
- ❌ Hard-code translated strings in tests
- ❌ Use CSS class selectors that may change
- ❌ Test translation accuracy (that is a separate concern)

## See also

- Playwright e2e testing (authoring) index