---
name: paniolo-playwright-test-best-practices
description: |
  Playwright E2E test conventions for this project — AAA pattern, translation-aware selectors, hydration waits, mock auth, staging DB sessions, two-user flows, debugging, and CI setup. Use when authoring or editing any Playwright spec, test helper, or E2E configuration. Do NOT use for unit tests or hook tests — load paniolo-vitest-test-best-practices instead.
license: MIT
metadata:
  version: 0.1.1
tags:
- playwright
- testing
user-invocable: true
references:
- references/plan-acceptance-evidence.md
- references/playwright-aaa-pattern.md
- references/playwright-debugging.md
- references/playwright-e2e-fixture-contract-validation.md
- references/playwright-failure-classification.md
- references/playwright-hydration-waits.md
- references/playwright-mock-auth-in-tests.md
- references/playwright-mock-auth-troubleshooting.md
- references/playwright-multiple-users-mock.md
- references/playwright-native-runner-process-ownership.md
- references/playwright-session-expiry.md
- references/playwright-staging-db-setup.md
- references/playwright-test-organization.md
- references/playwright-test-owned-entities.md
- references/playwright-test.md
- references/playwright-translation-aware-tests.md
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

## Key Rules

- **AAA pattern** — every test must have `// Arrange`, `// Act`, `// Assert` comments. For
  multi-step flows, use inline phase comments rather than collapsing everything.
  [playwright-aaa-pattern](references/playwright-aaa-pattern.md)

- **Translation-aware selectors** — use `data-testid` and semantic `data-*` attributes; never assert
  on translated text strings.
  [playwright-translation-aware-tests](references/playwright-translation-aware-tests.md)

- **Hydration waits are a narrow exception** — prefer web-first assertions, but when a page has a
  known React hydration settle after navigation, use the shared `HYDRATION_WAIT_MS` pattern instead
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

- **Single-spec debugging** — read the package script before placing forwarded arguments. A literal
  `--` can become a Playwright positional separator instead of being consumed by the package
  manager. Confirm the banner reports the intended project, spec count, and worker count.
  [playwright-debugging](references/playwright-debugging.md)

- **Classify failures before changing product code** — check runner readiness, fixture validity,
  product behavior, then cleanup. Capture HTTP status and response body for failed API waits.
  [playwright-failure-classification](references/playwright-failure-classification.md)

- **Validate the fixture contract first** — inspect identifiers, dimensions, relationships,
  permissions, lifecycle state, ownership, and cleanup recognition before weakening product
  validation.
  [playwright-e2e-fixture-contract-validation](references/playwright-e2e-fixture-contract-validation.md)

- **Own native runner processes exactly** — record roots and ready-state descendants, use OS-temp
  logs, terminate only owned trees, and verify expected ports independently after cleanup. A missing
  PID registry alone is not cleanup evidence.
  [playwright-native-runner-process-ownership](references/playwright-native-runner-process-ownership.md)

- **Own test data completely** — use collision-resistant identifiers for every created entity,
  local `try/finally` cleanup, and global teardown as a failure-safe. Shared-state gates should pass
  twice consecutively.
  [playwright-test-owned-entities](references/playwright-test-owned-entities.md)

- **E2E util filenames** — `e2e/**/*.e2e-util.ts` that default-export a single function use
  **camelCase** basenames matching the export (for example `runEffect.e2e-util.ts`), not kebab-case.
  [playwright-test-organization](references/playwright-test-organization.md)

## Output Format

Write code changes directly. After edits, output a brief bullet list of which conventions were
applied and which validation commands were run.

## Error Handling

- If the repository's lint command fails after changes, report it verbatim and fix scoped findings
  before declaring success.
- If a test fails due to a missing session file, instruct the user to run the matching
  `e2e:create-session:*` command rather than attempting to fix it in code.

## Validation

```bash
<your e2e test command> path/to/file.spec.ts  # targeted spec
<your e2e test command>                        # full suite
<your lint command>                            # lint check
```

## Skill Handoffs

- Unit or hook tests → load [the Vitest skill](../paniolo-vitest-test-best-practices/SKILL.md).
- TypeScript spec errors → load [the TypeScript skill](../paniolo-typescript-best-practices/SKILL.md).

## Do Not

- Do not use for unit tests or hook tests — load `paniolo-vitest-test-best-practices` instead.
- Do not add `waitForTimeout` arbitrary sleeps — use web-first assertions instead.
- Do not assert on translated text strings — use `data-testid` and semantic `data-*` attributes.
- Do not add polling loops in non-test code to fix flaky Realtime tests — fix the subscription.

## References

- Full reference: [playwright-test](references/playwright-test.md)
- Repo-wide rules: rules
- [Vitest skill](../paniolo-vitest-test-best-practices/SKILL.md)
- [TypeScript skill](../paniolo-typescript-best-practices/SKILL.md)
