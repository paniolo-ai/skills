---
name: paniolo-merge-ops
description: |
  Choose merge strategy, execute 'ship it' sequences, release worktree slots, pull latest main across checkouts, and refresh idle pool slots. Mutates git state (merges PRs, switches branches, and fast-forwards checkouts). Use when merging a PR, asked to 'ship it', or returning checkouts to clean main. Do NOT use for general git branching or worktree claiming; load `paniolo-worktree` for that.
license: MIT
metadata:
  version: 0.1.0
tags:
- git
- merge
- worktrees
- ship-it
references: []
---

**Requires:** terminal (`gh`, `git`, `paniolo`).

# Merge Operations

## Use When

- Merging a pull request, or being asked to "merge it".
- Executing a "ship it" request from the user.
- Returning checkouts and worktree pool slots to clean `main`.

---

## Merge Strategy

1. **Merge with a merge commit:** `gh pr merge <number> --merge`. Pass the strategy flag explicitly; never rely on defaults or the repository's allowed-methods setting.
2. **Do not squash-merge and do not rebase-merge.** Project history requires `Merge pull request #N` commits that preserve individual commit ancestry and build cache keys.
3. **Delete the remote topic branch:** Pass `--delete-branch` with the merge command unless another dependent PR explicitly builds on top of that branch.
4. **Merge only on request.** Opening a PR does not authorize merging it; merge only when the user explicitly asks or says "ship it".
5. **No unrequested auto-merge:** Do not enable auto-merge unless specifically requested.

---

## "Ship It" Contract

"Ship it" is the user's standing request for the complete delivery sequence. It authorizes the push, PR creation, merge, slot release, primary checkout fast-forward, and pool refresh without intermediate stops. Stop and ask only if a step fails or a check is red.

### Execution Sequence

1. **Push & PR:** Push the topic branch (`git push -u origin <branch>`) and open the PR (`gh pr create`).
2. **Merge:** Merge with a merge commit: `gh pr merge <number> --merge --delete-branch`.
3. **Release Slot:** If working in a claimed worktree pool slot (`repos/.dev-worker-*`), release it:
   ```bash
   paniolo worktree pool release --repo <repo> --path <slot-path> --owner <session-id>
   ```
4. **Fast-Forward Primary Checkout:** In the repository's primary checkout (`repos/<repo>`):
   ```bash
   git switch main
   git pull --ff-only origin main
   git branch -d <branch>
   ```
5. **Refresh Idle Pool Slots:** Fast-forward all idle slots in the pool to the refreshed base ref (`origin/main`):
   ```bash
   paniolo worktree pool refresh --repo <repo>
   ```
6. **Clean Harness Checkout:** Ensure the harness repository checkout returns to clean `main`:
   ```bash
   git switch main
   git pull --ff-only origin main
   ```

Report the PR number, merge commit hash, and confirmed state of the primary checkout and pool slots.

---

## Older Squash Merges

If a squash merge was previously performed outside this policy, leave `main` as is. Do not force-push or reset `main` to undo it unless explicitly instructed.

---

## References

- [paniolo-worktree](../paniolo-worktree/SKILL.md) — worktree pool lifecycle and commands
- [paniolo-safety-and-git](../paniolo-safety-and-git/SKILL.md) — repository safety guardrails and git conventions

## Do Not

- Do not pass `--squash` or `--rebase` to `gh pr merge`.
- Do not leave the primary checkout (`repos/<repo>`) or harness checkout on a topic branch after shipping.
- Do not omit `paniolo worktree pool refresh` after releasing a slot — idle slots must track latest `main`.
- Do not force-delete (`git branch -D`) an unmerged branch without explicit confirmation.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
