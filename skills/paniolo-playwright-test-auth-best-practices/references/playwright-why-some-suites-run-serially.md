---
source-wiki: sharp-shooter-wiki
source-slug: playwright-why-some-suites-run-serially
source-hash: f0aaa2caedbc8bc52f521a948efa86998d79295fb8e6f70693a101105d6fc63e
bundled: 2026-07-20
title: Why Some Suites Run Serially
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

# Why Some Suites Run Serially

Several specs use shared real users and shared staging data and are intentionally configured to
avoid parallel collisions — for example, sharing/invitation tests and realtime multi-user tagging
flows. That is why commands like `test:e2e:dev:staging-db` use `--workers=1`.

## See also

- Playwright auth testing (authoring) index