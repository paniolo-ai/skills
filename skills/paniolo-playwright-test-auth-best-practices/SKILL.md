---
name: paniolo-playwright-test-auth-best-practices
description: |
  Playwright authentication strategies for this project — mock auth helpers (authenticateTestUser, mockSignedOutUser, createTestUser), real signed-session flows for staging-DB and deployed runs, two-user sharing tests, and session management. Use when authoring or editing any Playwright spec that involves sign-in, sign-out, protected routes, multi-user flows, or staging DB sessions. Pair with playwright-test-best-practices for general E2E conventions.
license: MIT
metadata:
  version: 0.1.0
tags:
- playwright
- testing
- auth
references:
- 'wiki: sharp-shooter-wiki:playwright-auth → references/playwright-auth.md'
- 'wiki: sharp-shooter-wiki:playwright-authentication-model → references/playwright-authentication-model.md'
- 'wiki: sharp-shooter-wiki:playwright-mock-auth-api-reference → references/playwright-mock-auth-api-reference.md'
- 'wiki: sharp-shooter-wiki:playwright-mock-auth-best-practices → references/playwright-mock-auth-best-practices.md'
- 'wiki: sharp-shooter-wiki:playwright-mock-auth-in-tests → references/playwright-mock-auth-in-tests.md'
- 'wiki: sharp-shooter-wiki:playwright-mock-auth-troubleshooting → references/playwright-mock-auth-troubleshooting.md'
- 'wiki: sharp-shooter-wiki:playwright-multiple-users-mock → references/playwright-multiple-users-mock.md'
- 'wiki: sharp-shooter-wiki:playwright-session-expiry → references/playwright-session-expiry.md'
- 'wiki: sharp-shooter-wiki:playwright-staging-db-setup → references/playwright-staging-db-setup.md'
- 'wiki: sharp-shooter-wiki:playwright-test → references/playwright-test.md'
- 'wiki: sharp-shooter-wiki:playwright-why-some-suites-run-serially → references/playwright-why-some-suites-run-serially.md'
---

**Requires:** file-read, terminal. No network access needed.

**Full reference:**
[playwright-auth](references/playwright-auth.md)

**General Playwright reference:**
[playwright-test](references/playwright-test.md)

## Preconditions

- Read the target spec and any nearby auth helpers before editing.
- Check rules for repo-wide constraints.

## Defaults (proceed without asking)

- Apply all key rules below; edit the file already open or mentioned.
- **Always ask:** which spec file if not specified and cannot be inferred.

## Auth strategy decision tree

```text
Does the test need Realtime subscriptions, actual DB rows, or RLS enforcement?
├─ No  → Use mock auth helpers (authenticateTestUser / mockSignedOutUser)
└─ Yes → Use real signed sessions (storageState + staging DB setup)
             └─ Two users needed?
                 ├─ No  → Single context with GOOGLE*USER*SESSION_PATH
                 └─ Yes → Two contexts: GOOGLE*USER*SESSION*PATH + GOOGLE*USER*SESSION*PATH_2
```

[playwright-authentication-model](references/playwright-authentication-model.md)

## Key rules

- **Mock auth before navigation** — call `authenticateTestUser(page)` before `page.goto()`; never
  after. Race conditions result otherwise.
  [playwright-mock-auth-in-tests](references/playwright-mock-auth-in-tests.md)

- **Separate contexts for multiple users** — use `browser.newContext()` per user, never share a
  context. Wrap with `try/finally` and close both contexts.
  [playwright-multiple-users-mock](references/playwright-multiple-users-mock.md)

- **Test both auth states** — for any protected route, cover the authenticated _and_
  unauthenticated paths in the same `test.describe` block.
  [playwright-mock-auth-best-practices](references/playwright-mock-auth-best-practices.md)

- **Real sessions for Realtime/RLS tests** — use `test.use({ storageState })` with pre-signed
  cookies instead of mocking `/api/me` when tests need actual DB rows or Supabase Realtime.
  [playwright-staging-db-setup](references/playwright-staging-db-setup.md)

- **Session expiry** — JWT expires after 7 days; re-run the matching `e2e:create-session:*`
  command when you see `401 Not authenticated`.
  [playwright-session-expiry](references/playwright-session-expiry.md)

- **Serial execution for shared staging data** — two-user and realtime specs use
  `--workers=1` to prevent parallel collisions.
  [playwright-why-some-suites-run-serially](references/playwright-why-some-suites-run-serially.md)

- **Route mock pattern** — `**/api/me` matches both `http://localhost:8787/api/me` and
  `https://localhost:5173/api/me`; verify the pattern when a mock route is not intercepting.
  [playwright-mock-auth-troubleshooting](references/playwright-mock-auth-troubleshooting.md)

## Mock auth API (quick reference)

| Helper                       | What it does                                     |
| ---------------------------- | ------------------------------------------------ |
| `authenticateTestUser(page)` | Mocks `/api/me` → 200 with `DEFAULT*TEST*USER`   |
| `mockSignedOutUser(page)`    | Mocks `/api/me` → 401                            |
| `createTestUser(overrides?)` | Returns a `MockUserSession` merged over defaults |

[playwright-mock-auth-api-reference](references/playwright-mock-auth-api-reference.md)

## Session commands (quick reference)

```bash
# Create / refresh real sessions
npm run e2e:create-session:staging-db         # user 1, local IP
npm run e2e:create-session:staging-db:user2   # user 2, local IP
npm run e2e:create-session:staging-url        # user 1, public IP

# Run tests with real sessions
npm run test:e2e:dev:staging-db               # all specs, staging DB
npm run test:e2e:dev:staging-db:file -- <spec> --project=chromium  # single spec
```

[playwright-staging-db-setup](references/playwright-staging-db-setup.md)

## Examples

- Example spec:
  [example.spec.ts](/.agents/skills/playwright-test-auth-best-practices/example.spec.ts)

## Output format

Write code changes directly. After edits, output a brief bullet list of which
auth conventions were applied and which validation commands were run.

## Error handling

- If `npm run lint` fails after changes, report verbatim and fix before declaring success.
- If a test fails with `401 Not authenticated`, instruct the user to re-run the matching
  `e2e:create-session:*` command — do not attempt to fix it in code.
- If `storageState` file is missing, instruct the user to run the session-creation command
  for the relevant mode (staging-db or staging-url).

## Validation

```bash
npm run lint
```

## Skill handoffs

- General E2E patterns (AAA, selectors, hydration waits, debugging) →
  load `playwright-test-best-practices`.
- Unit or hook tests → load `vitest-test-best-practices` or
  `vitest-test-hook-best-practices`.
- TypeScript errors in spec files → load `paniolo-typescript-best-practices`.

## Do Not

- Do not call `authenticateTestUser(page)` after `page.goto()` — mock auth must precede navigation.
- Do not share a browser context between multiple users — use `browser.newContext()` per user.
- Do not use mock auth for tests that require actual DB rows, RLS enforcement, or Realtime.
- Do not attempt to fix a `401 Not authenticated` error in code.
  Re-run the session-creation command.
- Do not use for general E2E patterns — load `playwright-test-best-practices` instead.

## References

- Full auth reference: [playwright-auth](references/playwright-auth.md)
- General Playwright reference: [playwright-test](references/playwright-test.md)
- Repo-wide rules: rules
- [playwright-test-best-practices/SKILL.md](../paniolo-playwright-test-best-practices/SKILL.md)
- [vitest-test-best-practices/SKILL.md](../paniolo-vitest-test-best-practices/SKILL.md)
- [paniolo-typescript-best-practices/SKILL.md](../paniolo-typescript-best-practices/SKILL.md)