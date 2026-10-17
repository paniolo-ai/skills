---
source-slug: playwright-session-expiry
source-hash: c1f49d3d5d841156d4cbee140d3774a3346aa2d74447b93a72f2d88457b69ab5
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
