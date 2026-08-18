---
source-wiki: sharp-shooter-wiki
source-slug: playwright-debugging
source-hash: d5f105ca58afe6275284f0d292383d4b53582f7300d72ac9b1bae8b78f5c547b
bundled: 2026-08-17
title: Debugging
type: concept
tags:
- authoring
- playwright
- testing
- e2e
updated: 2026-06-18
---

# Debugging

Recommended order:

1. Run one spec, one browser with verbose browser console output:

   ```bash
   PLAYWRIGHT_VERBOSE=true npm run test:e2e:dev:staging-db:file -- e2e/specs/sharing/song-sharing.spec.ts --project=chromium 2>&1 > out.txt
   ```

   `PLAYWRIGHT_VERBOSE=true` enables full verbose output: every browser `console.*` call (prefixed
   with `[label:type]`, e.g. `[viewer:error]`, `[presenter:log]`), `e2eDebug(…)` calls in test
   files, and the wrangler env-var bindings table. Without it, all of those are suppressed to keep
   output readable.

2. Fix the narrow failure first.
3. Re-run that same spec.
4. Broaden to the surrounding suite.
5. Re-run the full command only after the focused case is stable.

When a test fails, check:

- Did the wrapper reach `PLAYWRIGHT_WRAPPER: READY`?
- Did the failure happen in Chromium only, or in Firefox/WebKit too?
- Was the failure a real assertion, a timeout, or a connection refusal?
- Do `/tmp/playwright-dev-client.log` and `/tmp/playwright-dev-api.log` show a server-side error?

## See also

- Playwright e2e testing (authoring) index
