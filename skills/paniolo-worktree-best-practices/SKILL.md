---
name: paniolo-worktree-best-practices
description: |
  Git worktree discipline for repositories several agents share at once. Use before running `git worktree add`, before committing in a checkout whose working tree holds changes you did not make, when a commit lands on the wrong branch, when a checkout fails on Windows path length, and when deleting a worktree or a merged branch. Do NOT use for ordinary branching in a repository you are the only session touching.
license: MIT
metadata:
  version: 0.1.0
tags:
- git
- worktrees
references:
- references/git-worktree-creation-and-cleanup.md
- references/git-worktree-shared-checkout-hazards.md
---

## First: do you need one?

A worktree earns its cost when **another session may be using the shared checkout**. `HEAD`, the branch set, and the stash stack are shared across every worktree of a repository, so a concurrent checkout moves your branch and a concurrent branch switch deletes your uncommitted files.

Decide from the checkout, not from preference:

- `git worktree list` shows worktrees you did not create → likely yes.
- Uncommitted changes you did not make, and the set grows between two `git status` calls → yes, a session is actively writing.
- One commit you can make and push immediately → no, the window is too small.

Full signal list and the loss modes behind each: `references/git-worktree-shared-checkout-hazards.md`.

## Always, regardless

- `git branch --show-current` **immediately before every commit**, not at the start of the task.
- Get your work onto your own branch early. Work held only in a shared working tree is one `git checkout` away from gone.
- Never bare `git stash` or `git stash pop` — the stack is shared. Prefer a temporary WIP commit.

## Never touch another session's work

Do not commit, stash, revert, or repair changes you did not make — not even a one-line fix in a file you are already editing. If a gate fails on their code, stop and show the output. Verify your staged set before every commit:

```bash
git diff --cached --name-status   # exactly your files, nothing else
```

## Creating one

Place it as a **sibling of the real checkout**, at the same depth. Hooks that
find a project root by climbing a fixed number of directories break anywhere
else, and the error names a directory you did not choose.

On Windows, check out only the subtree you need, or a long tracked filename
plus a deep worktree path exceeds 260 characters and leaves a registered
worktree with no usable index:

```bash
git worktree add --no-checkout <path> -b <branch> origin/main
cd <path> && git sparse-checkout init --cone
git sparse-checkout set <dir> && git checkout
```

`git worktree move` preserves sparse config and staging, so a worktree at the
wrong depth can be relocated rather than rebuilt.

## Before trusting a gate

A hook that validates a *configured* path rather than the current directory
grades the original checkout, not your worktree. Confirm which path it targeted
before treating a pass as evidence, and check that the configured checkout is
clean first — otherwise their unfinished work fails your commit. Never reach
for `--no-verify`.

## Cleanup

**Every step here is irreversible. Review before each one**: confirm the branch
is merged and the directory is yours, and prefer a tool that previews by
default — run the dry run, read the plan, then apply.

Rescue first: a branch may hold an unpushed commit that is not yours. Name a
branch at that SHA before deleting anything.

Squash-merged branches need `git branch -D`, because their commits are not
ancestors of the default branch — verify the content landed with
`git ls-tree origin/main -- <path>` first. For the same reason, a commit pushed
to a branch *after* its squash merge is not in that merge; cut a fresh branch
from the new default branch.

Change directory out of the worktree before removing it, verify removal by
checking the path rather than the exit code, and `git worktree prune` after any
manual deletion. Full sequence and failure modes:
`references/git-worktree-creation-and-cleanup.md`.

## Do Not

- Do not commit without checking the current branch first.
- Do not leave work only in a shared working tree.
- Do not use bare `git stash`.
- Do not full-checkout a large repository's worktree on Windows.
- Do not report a gate that validated a different checkout as validation of
  your worktree.
- Do not discard history to undo a misplaced commit. Name a branch at the
  commit, then `git branch -f <branch> origin/<branch>`.
- Do not fix, stash, or commit another session's in-flight work.
- Do not delete a worktree or force-delete a branch without first confirming it
  is merged and that no unpushed commit on it is someone else's.

## References

- `references/git-worktree-shared-checkout-hazards.md` — shared `HEAD`, the
  branch set and the stash stack; how uncommitted work is lost; when to take a
  worktree.
- `references/git-worktree-creation-and-cleanup.md` — hook path assumptions,
  Windows path length and sparse checkout, scratch build caches, and the full
  deletion sequence.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
