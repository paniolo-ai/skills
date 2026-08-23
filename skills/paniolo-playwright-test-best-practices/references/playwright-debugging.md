---
source-slug: playwright-debugging
source-hash: 52412a1e2f25edd98a56b40ae418e51ffcb21a778b22ac1856ac5cdb63f62511
bundled: 2026-08-20
title: Debugging
type: concept
tags:
- authoring
- playwright
- testing
- e2e
updated: 2026-08-20
---

# Debugging

Isolate one Playwright failure and classify it before changing application behavior. Start with the
repository's native package script so the documented environment and runner lifecycle still apply.

## Focused Windows Run

In PowerShell, set environment variables through PowerShell and invoke the package script directly:

```powershell
$env:PLAYWRIGHT_VERBOSE = "true"
pnpm run <single-spec-script> --project=chromium path/to/example.spec.ts
Remove-Item Env:PLAYWRIGHT_VERBOSE
```

Read the package script before adding a literal `--`. Some wrappers already forward arguments, so
that token can change Playwright's project or file selection. Confirm the banner reports the
intended project, spec count, and worker count.

Use the wrapper's operating-system temporary directory for logs and PID metadata. Do not assume
that `/tmp` exists on Windows, and do not depend on shell redirection for required diagnostics.

## Diagnosis Order

1. Confirm the wrapper printed its readiness marker and intended test selection.
1. Validate the session, roles, identifiers, relationships, and domain values in the fixture.
1. Find the first failed assertion or failed request with its response status and body.
1. Inspect owned server logs for an earlier child exit or backend error.
1. Confirm teardown removed owned rows, descendants, and listeners.
1. Re-run the focused case before broadening to the surrounding suite.

An outer timeout, forced process termination, or reporter pipe error is inconclusive unless a test
assertion already established a product failure.

## See Also

- E2E Fixture Contract Validation
- Playwright Failure Classification
- Native Runner Process Ownership
- Playwright e2e testing (authoring) index
