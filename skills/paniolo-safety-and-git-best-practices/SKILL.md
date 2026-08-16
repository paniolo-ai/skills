---
name: paniolo-safety-and-git-best-practices
description: |
  Repository-wide safety guardrails for command execution, environment management, git usage, database migrations, and honest completion reporting. Use before running deploys or migrations, before git write operations, and before reporting a task complete. Do NOT use for code-authoring conventions — load the language skill instead.
license: MIT
metadata:
  version: 0.2.0
tags:
- git
- safety
references:
- 'wiki: sharp-shooter-wiki:applied-migrations-are-immutable → references/applied-migrations-are-immutable.md'
- 'wiki: sharp-shooter-wiki:completion-means-the-gate-ran → references/completion-means-the-gate-ran.md'
---

## Command Execution Safety

- Safe defaults:
  - `npm run test:unit` (targeted files preferred)
  - `npm run lint`
  - `npm run lint:md`
  - `npm run format`
- Never auto-run without explicit user ask:
  - git write operations
  - deployments or production migrations
  - system-level package installs

## Environment Safety

- Staging-first for deploys and migrations.
- Production deploy/migration commands require explicit user confirmation.
- **Read the script definition before running a migration.** Environment-suffixed commands are
  explicit (`supabase:migrate:staging`); the **unsuffixed one is frequently production**. Do not
  infer the target from the name.

## Migration Safety

- **A migration applied to any shared environment is immutable.** Correct it by adding a new
  migration, never by editing the file — including changes that alter no SQL, such as reworded
  comments or reformatting. Tooling tracks applied state by filename and content checksum.
- When unsure whether a migration has been applied, assume it has.
  [applied-migrations-are-immutable](references/applied-migrations-are-immutable.md)

## Reporting Completion

- **A task is complete when its verification command has been run and passed** — not when the code
  looks right or matches the plan. If the suite was not run, say so; "implemented, tests not run" is
  a useful status, "complete" is a false one.
- **Changing shared or derived data means running the consumers' tests.** Adding a defaulted field
  to a normalizer, decoder, or fixture changes the shape every caller sees; that module's own test
  passing proves nothing about callers asserting the old shape. Scope the run by what the change
  reaches, not by what you edited.
- Plan and checklist markers describe what landed, not what was attempted.
- If a test fails and you believe it is a flake, say it failed and why, then re-run it in isolation.
  [completion-means-the-gate-ran](references/completion-means-the-gate-ran.md)

## Git Usage

- No git write operations by default.
- Read-only git commands are allowed.
- Do not prompt users to commit or open PRs unless they explicitly request PR help.

## Do Not

- Do not auto-run git write operations (commit, push, branch, reset, rebase).
- Do not run production deployments or migrations without explicit user confirmation.
- Do not install system-level packages without user approval.
- Do not prompt to commit or open a PR unless the user explicitly requests it.
- Do not edit a migration that has been applied to any shared environment — add a new one.
- Do not report work complete without having run its verification command and seen it pass.
- Do not mark a plan or checklist item done for work that has not landed.
- Do not stay silent about a failing test, including one you believe is a flake.

## References

- Repo-wide rules: rules
- [applied-migrations-are-immutable](references/applied-migrations-are-immutable.md)
- [completion-means-the-gate-ran](references/completion-means-the-gate-ran.md)