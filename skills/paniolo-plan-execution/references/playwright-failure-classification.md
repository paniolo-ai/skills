---
source-slug: playwright-failure-classification
source-hash: c550208d24f885dbd23fd93e1273477c024efd3e582d299a1c91c8a0a6dfd9a7
bundled: 2026-08-20
title: Playwright Failure Classification
type: concept
tags:
- authoring
- playwright
- testing
- debugging
updated: 2026-08-20
---

# Playwright Failure Classification

Classify a failing end-to-end run before changing application behavior. Debug in this order:

1. **Runner readiness** — did the expected servers start, did the readiness marker appear, and did
   the intended project/spec count run?
1. **Fixture validity** — are sessions, roles, IDs, domain values, and prerequisite rows valid for
   the behavior under test?
1. **Product behavior** — did the request or interaction violate the implemented contract?
1. **Cleanup and repeatability** — did prior rows or processes affect the run?

This order prevents infrastructure and fixture defects from being disguised as product changes.
Do not add application polling, refetch loops, retries, or relaxed validation merely to make an
invalid test pass.

## Evidence to capture

- The printed Playwright project, spec count, and worker count.
- Server readiness output and owned-process logs.
- HTTP status and response body for failed API waits.
- The exact fixture values involved in domain validation.
- Teardown counts and remaining owned processes or ports.

If an outer timeout kills an otherwise unfinished test process, report the gate as inconclusive. A
reporter `EPIPE` after forced termination describes the broken output pipe, not a failed assertion.

## See also

- Debugging
- E2E Fixture Contract Validation
- Native Runner Process Ownership
- Test-Owned Entities
