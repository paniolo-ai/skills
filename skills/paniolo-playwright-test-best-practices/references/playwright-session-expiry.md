---
source-wiki: sharp-shooter-wiki
source-slug: playwright-session-expiry
source-hash: 8b6bdf45112636f778b798f1012d79cdbbe0758e7b2158429bafb6013a168c5b
bundled: 2026-08-17
title: Session Expiry
type: concept
tags:
- authoring
- playwright
- testing
- e2e
- auth
updated: 2026-06-18
---

# Session Expiry

The JWT expires after **7 days**. When you see `401 Not authenticated` errors, regenerate:

```bash
# Mode 1 (local site)
npm run e2e:create-session:staging-db

# Mode 2 (staging site)
npm run e2e:create-session:staging-url
```

The file path stays the same so no spec changes are needed.

## See also

- Playwright auth testing (authoring) index
