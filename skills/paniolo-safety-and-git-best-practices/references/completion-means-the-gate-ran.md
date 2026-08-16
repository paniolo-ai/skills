---
source-wiki: sharp-shooter-wiki
source-slug: completion-means-the-gate-ran
source-hash: 4e96fc153fa0330b8dc65bac6be606c0e594bd9c0a5e5dc54e1b748e81d12061
bundled: 2026-08-15
title: Completion Means the Gate Ran
type: concept
tags:
- authoring
- agents
- testing
- safety
updated: 2026-08-15
---

# Completion Means the Gate Ran

A task is complete when its verification command has been **run** and **passed**. Not when the code
looks right, not when the diff matches the plan, and not when the parts you thought to check pass.

This is the most expensive failure an agent can produce, because it is the one that removes the
reader's ability to trust everything else. A defect that is reported honestly costs a fix. A defect
inside work reported as finished costs a review of the whole batch.

## Two rules

### 1. Report status from the command's output, never from intent

If the suite was not run, say so. "Implemented; tests not yet run" is a useful, honest status.
"Complete" for unrun work is a false one, and it is indistinguishable from success until someone
else pays to discover otherwise.

When a plan or checklist tracks progress, its markers describe **what landed**, not what was
attempted. Marking a card done for work that does not exist turns the plan from a record into a
second thing that has to be verified.

### 2. Changing shared data means running the *consumers'* tests

Adding a defaulted field to a normalizer, decoder, factory, or fixture changes the shape every
caller sees. Updating that module's own test proves the module works; it proves nothing about the
callers asserting on the old shape.

This is precisely how a failing test gets committed by someone who did run tests — they ran the
ones they were thinking about. Scope the run by **what the change reaches**, not by what you edited:

```bash
# The changed file's own tests — necessary, not sufficient
vitest run src/user/normalizeUserRow.test.ts

# Everything that decodes, asserts on, or snapshots that shape
vitest run src/user src/session src/live
```

## Practical guidance

- Prefer a scoped run over no run. A directory-level run that finishes in a minute beats a full
  suite you skip because it takes twenty.
- Run the full gate — typecheck, lint, tests — before declaring a batch of work finished, even when
  each individual change was verified.
- When a test fails and you believe it is a flake, **say that it failed and why you think so**, then
  re-run it in isolation. Silence about a red test is the same error as never running it.
- A pre-commit hook is a backstop, not the verification step. Work handed over before the hook runs
  has not been verified.

## See also

- [Update Guards Cover Every Gated Field](typescript-update-guards-cover-every-gated-field.md)
- [Authoring](index.md)
