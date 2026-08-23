---
source-slug: playwright-test-owned-entities
source-hash: 645603834e35b138de1a7da35cde4493a3eea32cd6acc3ecb525686257e77704
bundled: 2026-08-20
title: Test-Owned Entities
type: concept
tags:
- authoring
- playwright
- testing
- e2e
updated: 2026-08-20
---

# Test-Owned Entities

Playwright specs should create the domain entities they assert on. Prefer setup through the same
API or UI surface the product uses, then remove the records through the test flow or a focused
cleanup helper.

Use a collision-resistant identifier that the repository's cleanup contract recognizes:

```text
e2e-<entity>-<run-id>-<random-suffix>
```

Do not make normal Playwright commands depend on shared mutable seed scripts. Manual repair or seed
scripts can support one-off environment maintenance, but they are not fixtures for the behavior
under test.

Every entity class created by a spec needs an owned identifier and cleanup path, not only the
top-level record. Register dependent records as they are created and remove them in reverse
dependency order inside `try/finally`. Close additional browser contexts in the same `finally`
block.

Keep global teardown as a failure-safe for interrupted workers; it does not replace local cleanup.
Teardown should report how many owned records it removed so zero or unexpectedly high counts stay
visible. Shared-state release gates should pass twice consecutively to prove that the second run
does not consume leftovers from the first.

Repositories must define their exact disposable prefixes, protected fixtures, owner boundaries,
and cascade behavior in repo-local guidance. Generic skills must not guess those values.

## See Also

- E2E Fixture Contract Validation
- Playwright Failure Classification
- Playwright e2e testing (authoring) index
