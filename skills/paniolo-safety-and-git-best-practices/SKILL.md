---
name: paniolo-safety-and-git-best-practices
description: |
  Repository-wide safety guardrails for command execution, environment management, and git usage.
license: MIT
metadata:
  version: 0.1.0
tags:
- git
- safety
references: []
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

## Git Usage

- No git write operations by default.
- Read-only git commands are allowed.
- Do not prompt users to commit or open PRs unless they explicitly request PR help.

## Do Not

- Do not auto-run git write operations (commit, push, branch, reset, rebase).
- Do not run production deployments or migrations without explicit user confirmation.
- Do not install system-level packages without user approval.
- Do not prompt to commit or open a PR unless the user explicitly requests it.

## References

- Repo-wide rules: rules