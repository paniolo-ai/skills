---
source-wiki: sharp-shooter-wiki
source-slug: playwright-test-organization
source-hash: 4eb177973aca85d0d89beffb07ba6f2e4ffc292ed9405341d441b61213a71ae0
bundled: 2026-08-17
title: Test Organization
type: concept
tags:
- authoring
- playwright
- testing
- e2e
updated: 2026-06-18
---

# Test Organization

Tests live under `e2e/`:

- `e2e/specs/` — feature specs and user-flow specs
- `e2e/specs/sharing/helpers/` — two-user share/invitation helpers
- `e2e/specs/tagging/helpers/` — realtime tagging helpers
- `e2e/utils/` — auth setup, response wait helpers, effect wrappers, console/error tracking

**`*.e2e-util.ts` naming:** when a helper file **default-exports one function**, the basename is
**camelCase** and matches that symbol (for example `createPublicLiveCommunityChain.e2e-util.ts` in
`e2e/utils/live/`). Kebab-case basenames are for multi-export or non-function modules only.

Test styles used in this repo:

- Anonymous/public page tests
- Authenticated single-user tests
- Two-user real-session sharing/invitation tests
- Realtime cross-user tests
- Accessibility checks with `@axe-core/playwright`
- Lighthouse performance audits

## See also

- Playwright e2e testing (authoring) index
