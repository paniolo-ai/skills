---
source-slug: playwright-session-expiry
source-hash: e821001696a5d62aab958175a53d43a0f0567afa29e98a71772e099ac73ee449
bundled: 2026-08-20
title: Session Expiry
type: concept
tags:
- authoring
- playwright
- testing
- e2e
- auth
updated: 2026-08-20
---

# Session Expiry

The JWT expires after **7 days**. When you see `401 Not authenticated` errors, regenerate:

```bash
# Mode 1 (local site)
pnpm run e2e:create-session:staging-db

# Mode 2 (staging site)
pnpm run e2e:create-session:staging-url
```

The file path stays the same so no spec changes are needed.

## See also

- Playwright auth testing (authoring) index
