---
name: paniolo-playwright-test-best-practices
description: |
  Playwright E2E test conventions for this project — AAA pattern, translation-aware selectors, hydration waits, mock auth, staging DB sessions, two-user flows, debugging, and CI setup. Use when authoring or editing any Playwright spec, test helper, or E2E configuration. Do NOT use for unit tests or hook tests — load paniolo-vitest-test-best-practices instead.
license: MIT
metadata:
  version: 0.1.0
tags:
- playwright
- testing
references:
- 'wiki: sharp-shooter-wiki:playwright-test → references/playwright-test.md'
- 'wiki: sharp-shooter-wiki:playwright-aaa-pattern → references/playwright-aaa-pattern.md'
- 'wiki: sharp-shooter-wiki:playwright-translation-aware-tests → references/playwright-translation-aware-tests.md'
- 'wiki: sharp-shooter-wiki:playwright-hydration-waits → references/playwright-hydration-waits.md'
- 'wiki: sharp-shooter-wiki:playwright-mock-auth-in-tests → references/playwright-mock-auth-in-tests.md'
- 'wiki: sharp-shooter-wiki:playwright-multiple-users-mock → references/playwright-multiple-users-mock.md'
- 'wiki: sharp-shooter-wiki:playwright-mock-auth-troubleshooting → references/playwright-mock-auth-troubleshooting.md'
- 'wiki: sharp-shooter-wiki:playwright-staging-db-setup → references/playwright-staging-db-setup.md'
- 'wiki: sharp-shooter-wiki:playwright-session-expiry → references/playwright-session-expiry.md'
- 'wiki: sharp-shooter-wiki:playwright-debugging → references/playwright-debugging.md'
- 'wiki: sharp-shooter-wiki:playwright-test-organization → references/playwright-test-organization.md'
---

**Requires:** file-read, terminal. No network access needed.

**Full reference:**
[playwright-test](references/playwright-test.md)

## Preconditions

- Read the spec file before editing.
- Check rules for repo-wide constraints.

## Defaults (proceed without asking)

- Apply all key rules below; edit the file already open or mentioned.
- **Always ask:** which spec file if not specified and cannot be inferred.

## Key rules

- **AAA pattern** — every test must have `// Arrange`, `// Act`, `// Assert` comments. For
  multi-step flows, use inline phase comments rather than collapsing everything.
  [playwright-aaa-pattern](references/playwright-aaa-pattern.md)

- **Translation-aware selectors** — use `data-testid` and semantic `data-*` attributes; never assert
  on translated text strings.
  [playwright-translation-aware-tests](references/playwright-translation-aware-tests.md)

- **Hydration waits are a narrow exception** — prefer web-first assertions, but when a page has a
  known React hydration settle after navigation, use the shared `HYDRATION*WAIT*MS` pattern instead
  of ad hoc sleeps.
  [playwright-hydration-waits](references/playwright-hydration-waits.md)

- **Mock auth before navigation** — call `authenticateTestUser(page)` before `page.goto()`; never
  after.
  [playwright-mock-auth-in-tests](references/playwright-mock-auth-in-tests.md)

- **Separate contexts for multiple users** — use `browser.newContext()` per user; never share a
  context.
  [playwright-multiple-users-mock](references/playwright-multiple-users-mock.md)

- **Web-first assertions first** — use `await expect(...).toBeVisible()` with a timeout for UI
  readiness; do not add arbitrary `waitForTimeout` calls.
  [playwright-mock-auth-troubleshooting](references/playwright-mock-auth-troubleshooting.md)

- **Real sessions for Realtime/RLS tests** — use `storageState` with pre-signed cookies instead of
  mocking `/api/me` when tests need actual DB rows or Supabase Realtime.
  [playwright-staging-db-setup](references/playwright-staging-db-setup.md)

- **Never fix flakiness by polling in app source** — if a test flakes because a realtime update is
  slow or missing, the bug is in the Realtime/RLS setup, not the absence of a re-fetch loop. Fix
  the subscription or use a web-first wait in the test. Do **not** add `setInterval`/`setTimeout`
  polling to non-test code — it is banned
  (rules → "No polling in non-test code").

- **Session expiry** — JWT expires after 7 days; re-run the matching `e2e:create-session:*` command
  when you see `401 Not authenticated`.
  [playwright-session-expiry](references/playwright-session-expiry.md)

- **Single-spec debugging** — use `test:e2e:dev:staging-db:file -- <spec> --project=chromium` to
  isolate failures before broadening.
  [playwright-debugging](references/playwright-debugging.md)

- **E2E util filenames** — `e2e/**/*.e2e-util.ts` that default-export a single function use
  **camelCase** basenames matching the export (for example `runEffect.e2e-util.ts`), not kebab-case.
  [playwright-test-organization](references/playwright-test-organization.md)

## Output format

Write code changes directly. After edits, output a brief bullet list of which conventions were
applied and which validation commands were run.

## Error handling

- If `npm run lint` fails after changes, report verbatim and fix before declaring success.
- If a test fails due to a missing session file, instruct the user to run the matching
  `e2e:create-session:*` command rather than attempting to fix it in code.

## Validation

```bash
<your e2e test command> path/to/file.spec.ts  # targeted spec
<your e2e test command>                        # full suite
<your lint command>                            # lint check
```

## Skill handoffs

- Unit or hook tests → load `paniolo-vitest-test-best-practices`.
- TypeScript errors in spec files → load `paniolo-typescript-best-practices`.

## Do Not

- Do not use for unit tests or hook tests — load `paniolo-vitest-test-best-practices` instead.
- Do not add `waitForTimeout` arbitrary sleeps — use web-first assertions instead.
- Do not assert on translated text strings — use `data-testid` and semantic `data-*` attributes.
- Do not add polling loops in non-test code to fix flaky Realtime tests — fix the subscription.

## References

- Full reference: [playwright-test](references/playwright-test.md)
- Repo-wide rules: rules
- `paniolo-vitest-test-best-practices`
- `paniolo-typescript-best-practices`
