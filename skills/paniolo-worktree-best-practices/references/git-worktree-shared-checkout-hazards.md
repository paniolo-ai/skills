---
source-slug: git-worktree-shared-checkout-hazards
source-hash: e6cafaa3217b5bf09a204eeefb8f2179a427f1d18802ebfb5f5190cf06eef397
bundled: 2026-10-16
title: Git Worktree Shared Checkout Hazards
type: concept
tags:
- git
- worktrees
- concurrency
- agents
- safety
updated: 2026-10-11
---

# Git Worktree Shared Checkout Hazards

A repository checked out once and used by several agents at the same time behaves differently from one you have to yourself. Three pieces of state are shared across every worktree of a repository — `HEAD`, the branch set, and the stash stack — and each of them is a way to lose work that `git status` will not warn you about.

## `HEAD` moves under you

A concurrent `git checkout` in any worktree of the repository changes which branch is current. A commit you then make lands on whatever branch is current at that moment — someone else's feature branch, or the default branch.

The habit that catches the entire class:

```bash
git branch --show-current    # immediately before every commit
```

Not at the start of the task. Immediately before, because the window between
deciding to commit and committing is exactly when another session moves it.

## Uncommitted work in a shared tree is borrowed, not owned

When another session branches away, git carries over what it can and silently
drops the rest. Files that exist only in the working tree — new files, and
modifications to files that differ between the two revisions — are gone.

Observed on 2026-10-11: a concurrent session committed its work and switched
branches, clearing five new pages, two modified files, and three log entries
from the shared tree. The pages survived only because they had already been
committed on a separate branch.

So the rule is **get your work onto your own branch early, not at the end.** A
long editing session that keeps everything in the working tree until a final
commit is one `git checkout` away from losing all of it.

## A dirty tree can refuse to let you leave

The usual recipe for committing around someone else's work — branch off the
base, stage only your files, switch back — fails as soon as the other session
holds a modified file that differs between the two revisions:

```text
$ git checkout -b my-branch main
    wiki/plan-staleness-run-surface.md
Please commit your changes or stash them before you switch branches.
Aborting
```

Neither escape is acceptable: committing sweeps in their unfinished work, and
stashing takes it away from them mid-task. A separate worktree is the only
route left — see [git-worktree-creation-and-cleanup](./git-worktree-creation-and-cleanup.md).

## The stash stack is shared

`git stash` is per-repository, not per-worktree. A bare `git stash pop` can
restore another session's entry, and a bare `git stash` can bury yours under
theirs.

Prefer a temporary WIP commit on your own branch. If you must stash:

```bash
git stash push -u -m "unique-tag"
git stash list --format='%H %gs'     # capture YOUR entry's SHA
git stash apply <sha>                # apply, never pop
```

Then drop that entry specifically, re-finding its current `stash@{n}` by tag.

## Signals you need your own worktree

- `git worktree list` shows worktrees you did not create.
- Uncommitted changes you did not make — especially if the set grows between
  two `git status` calls minutes apart. That means a session is actively
  writing, not that someone left work behind.
- Work spanning many minutes where you will branch, commit, and push.

Against, even with concurrency: a single commit you can make and push
immediately, where the window is too small to matter.

## Never fix what you find

Another session's in-flight code is theirs to finish. Do not commit it, stash
it, revert it, or repair it to get a clean gate signal — not even a one-line
fix in a file you are already touching. If a gate fails on their work, stop and
show the output.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
