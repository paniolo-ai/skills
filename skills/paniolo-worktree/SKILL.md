---
name: paniolo-worktree
description: |
  Git worktree pool rules, slot lifecycle, and shared-checkout concurrency discipline. Covers pool claims, releases, per-slot ports, non-pool sibling worktrees, Windows path length, sparse checkouts, and harness single-checkout rules. Use when working in pool checkouts, creating git worktrees, or handling concurrent checkouts.
license: MIT
metadata:
  version: 0.4.0
tags:
- git
- worktrees
- worktree-pool
references:
- references/git-worktree-creation-and-cleanup.md
- references/git-worktree-shared-checkout-hazards.md
- references/worktree-slots.md
---

## First: Which Kind of Repository Is This?

Check `paniolo.config.json` for a `worktreePool` definition under the repository's configuration.

- **It has a pool:** Work only in a claimed slot. Never run `git worktree add` directly for it, and never edit its primary checkout.
- **It is the harness itself:** It gets no worktrees at all. Its primary checkout is slot `00`.
- **It has no pool:** Follow the plain Git worktree discipline detailed below.

## 1. Managed Pool Slot Discipline

### Claiming a Slot

```bash
paniolo worktree pool status --repo <key>
paniolo worktree pool claim --repo <key> --owner <session-id> --branch <topic>
```

- Always inspect state with `status` before claiming.
- Use your session ID as the owner; repeated claims with the same owner return your existing slot.
- Work only in the path returned by `claim`. Never guess a slot path from directory listings.
- To continue an open pull request branch, claim with `--from origin/<branch>`.
- If no slot is idle, wait. Never take over a slot owned by another session.

### Releasing a Slot

Release a slot only after its branch is pushed to origin or merged, and the working tree is clean:

```bash
paniolo worktree pool release --repo <key> --path <slot> --owner <session-id>
```

`release` refuses dirty slots to protect uncommitted work. Delete the local merged branch afterward with `git branch -d <branch>`.

### Slot Ports and Environment

Never hardcode ports or infer them from directory names. Query the slot environment:

```bash
paniolo worktree pool env --repo <key>
```

Use `PANIOLO_PORT_<NAME>` variables for test runs so test executions do not collide with active dev servers in other slots.

### Busy, Failed, or Abandoned Slots

- **Owned by someone else:** Never reset, force-release, or take over another agent's claim.
- **`failed` status:** Files and branches remain intact. Recover using `refresh` or `cleanup --retry-failed`.
- **Abandoned candidates:** `status --stale-hours 24` lists idle candidates. Use `reclaim --path <slot>` (dry run first, then `--apply`) only when the branch is pushed or merged and the checkout is clean.

## 2. Repositories Without a Pool: Plain Git Worktree Discipline

When multiple sessions touch a non-pool checkout, use a worktree to prevent shared `HEAD` switches from destroying uncommitted work:

- **Check current branch before committing:** Run `git branch --show-current` immediately before every commit.
- **Never touch another session's work:** Do not commit, stash, or revert changes you did not make. Verify staged files before committing:
  ```bash
  git diff --cached --name-status
  ```
- **Never bare stash:** Never use bare `git stash` or `git stash pop` in a shared checkout; the stash stack is shared across all worktrees.
- **Placement:** Always place worktrees as siblings of the primary checkout at the same directory depth so relative root traversals remain stable.
- **Windows path length and sparse checkouts:** On Windows, avoid MAX_PATH (260 char) errors by checking out only needed subtrees:
  ```bash
  git worktree add --no-checkout <path> -b <branch> origin/main
  cd <path> && git sparse-checkout init --cone
  git sparse-checkout set <dir> && git checkout
  ```
- **Updating a branch checked out elsewhere:** When adding to a branch currently checked out in a shared tree:
  ```bash
  git worktree add --detach <path> origin/<branch>
  # commit there, then push:
  git push origin HEAD:<branch>
  ```
- **Safe cleanup:** Confirm the branch is merged on origin before removal. Run `git worktree remove <path>` followed by `git worktree prune`.

## 3. Harness Repositories: Single Checkout Slot 00

Harness repositories validate against sibling satellites in `repos/`. A worktree of a harness has an empty `repos/` directory, so its gates cannot validate real code:

- Never create a worktree of a harness repository (`repos/.pr-*`, `.claude/worktrees/*`, etc.).
- Agents take turns in slot `00`. Inspect `git status` before starting. If another branch or uncommitted change is present, stop and coordinate.
- Branch from `main`, commit only your files, push, and open a PR. Never leave local commits on `main`.
- Always return the checkout to clean `main` before ending your turn.

## 4. Gate Validation in Worktrees

Harness commands such as `paniolo wiki`, `paniolo scan`, and `paniolo skills bundle` validate the checkout path declared in `paniolo.config.json`, not your worktree slot:

- Confirm gate output names your slot path before treating a pass as evidence.
- To validate a slot without dirtying tracked configuration, use a temporary config:

  ```powershell
  $c = Get-Content paniolo.config.json -Raw | ConvertFrom-Json
  $c.repos.'<repo>'.path = '<path to your slot>'
  $c | ConvertTo-Json -Depth 60 | Set-Content paniolo.config.tmp.json
  paniolo wiki --config paniolo.config.tmp.json --fail-on error
  Remove-Item paniolo.config.tmp.json
  ```

- Never use `--no-verify` to bypass pre-commit gates.

## 5. Where Worktrees Must Not Go

- Never place worktrees inside `.cache/`. Automated bootstrap commands clear `.cache` and delete active worktrees.
- The `.dev-worker-` directory prefix is strictly reserved for pool slots.

## Do Not

- Do not create a worktree for a repository managed by a worktree pool.
- Do not create a worktree for a harness repository.
- Do not take over, reset, or release a slot owned by another session.
- Do not release a slot whose branch has unpushed commits or uncommitted edits.
- Do not use bare `git stash` in shared repositories.
- Do not trust gate output that validated a checkout other than your own.
- Do not end a harness session on a topic branch.

## References

- [worktree-slots](references/worktree-slots.md)
- [git-worktree-shared-checkout-hazards](references/git-worktree-shared-checkout-hazards.md)
- [git-worktree-creation-and-cleanup](references/git-worktree-creation-and-cleanup.md)
- [paniolo-merge-ops](../paniolo-merge-ops/SKILL.md)

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
