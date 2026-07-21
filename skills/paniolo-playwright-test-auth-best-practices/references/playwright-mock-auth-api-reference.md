---
source-wiki: sharp-shooter-wiki
source-slug: playwright-mock-auth-api-reference
source-hash: e72309d0cd67a6bf0b007d2c72bc629ec2bb02ccb2bd7e4e75cfff551e66b220
bundled: 2026-07-20
title: Mock Auth API Reference
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

# Mock Auth API Reference

**`authenticateTestUser(page, userSession?)`** — mocks `/api/me` to return an authenticated user.
Defaults to `DEFAULT*TEST*USER`.

**`mockSignedOutUser(page)`** — mocks `/api/me` to return 401.

**`createTestUser(overrides?)`** — returns a `MockUserSession` with custom properties merged over
the defaults.

**`DEFAULT*TEST*USER`:**

```typescript
{
  user: {
    user_id: "test-user-id-12345",
    email: "test@example.com",
    name: "Test User",
    username: "testuser",
  }
}
```

## See also

- Playwright auth testing (authoring) index