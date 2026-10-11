---
source-slug: git-worktree-creation-and-cleanup
source-hash: f8e4d8a1ef17934163fe6fb4709e09efb867562bbf953cb7c287baf814ebb474
bundled: 2026-10-11
title: Git Worktree Creation and Cleanup
type: concept
tags:
- git
- worktrees
- windows
- cleanup
- agents
updated: 2026-10-11
---

# Git Worktree Creation and Cleanup

Where a worktree sits decides whether it works. Three constraints apply before
you have written a line, and the cleanup has its own traps.

## Hooks resolve paths relative to the repository, so depth matters

A hook that finds a project root by climbing a fixed number of directories
breaks in a worktree placed anywhere else. The failure names a directory you
did not choose:

```text
[ERR_PNPM_NO_IMPORTER_MANIFEST_FOUND] No package.json ... was found in "C:\Users".
```

That came from a `pre-commit` of `cd '../..' && exec pnpm run check:wiki`, which
resolves correctly from the repository's normal location and resolves to the
filesystem root from a scratch directory two levels deep.

Two consequences:

- **Place the worktree at the same depth as the real checkout** — a sibling of
  it, not in a temp directory.
- **Read the hook before assuming the gate ran.** Locally installed hooks are
  often untracked, so they are invisible in git history and only readable at
  `.git/hooks/`.

Beware the related trap: a hook that validates a *configured* path rather than
the current directory grades the original checkout, not your worktree. A pass
is then evidence about someone else's files. Check the configured checkout is
clean first, and never report such a pass as validation of worktree content.

## Long paths: check out only what you need

On Windows a full checkout can fail before it finishes, because a deep worktree
path plus a long tracked filename passes the 260-character limit:

```text
error: unable to create file raw/.../very-long-name-2026-07-19.md: Filename too long
fatal: Could not reset index file to revision 'HEAD'
```

The second line is the real damage. The worktree registers but has no usable
index, so `git worktree remove` then rejects it as *not a working tree* and
cleanup needs `rm -rf`, `git worktree prune`, and `git branch -D`.

Check out only the subtree you came for:

```bash
git worktree add --no-checkout <path> -b <branch> origin/main
cd <path>
git sparse-checkout init --cone
git sparse-checkout set <dir> [<dir>...]
git checkout
```

Skipping large unrelated subtrees also keeps a second copy of them out of any
workspace-wide search index.

`git worktree move` preserves both the sparse-checkout configuration and the
staging area, so a worktree created at the wrong depth can be relocated rather
than rebuilt — including after you have already staged files.

## Scratch build directories

A per-worktree build cache multiplies disk use by the number of worktrees, and
a cache keyed on the worktree path means a build in one place and a stale
binary in another. Set the build-output variable deliberately for ad-hoc
builds, and know that a path you invent is yours to delete: automated reclaim
scans a fixed naming pattern and cannot see a name outside it.

Prefer the main checkout for compiled-language work when it is free. A warm
cache turns a long cold build into a short one.

## Cleanup

Deletions are the only irreversible steps here.

1. **Rescue before deleting.** A branch may hold a commit that is not yours and
   not pushed. Create a named branch at that SHA first.
2. **Squash-merged branches need `git branch -D`.** Their commits are not
   ancestors of the default branch, so `-d` refuses them and
   `git log main..<branch>` still lists them even though the content landed.
   Verify the content with `git ls-tree origin/main -- <path>`, then force.
   For the same reason, a commit pushed to a branch *after* its squash merge is
   not in the merge — cut a fresh branch from the new default branch for it.
3. **Change directory out first.** `git worktree remove --force` fails with
   *permission denied* when the path is your shell's current directory.
4. **Verify removal by checking the path, not the exit code.** A shell profile
   that fails to initialize can abort the command on stderr while leaving the
   worktree in place. Observed with a Node version manager: two consecutive
   failures under one shell, clean removal under another.
5. `git worktree add -b <branch>` fails if the branch already exists; drop `-b`
   to reuse it.
6. `git worktree prune` after any manual directory deletion.

## Undo a misplaced commit without discarding history

Name a branch at the commit, then point the mistaken branch back at its
upstream:

```bash
git branch rescue/<name> <sha>
git branch -f <branch> origin/<branch>
```

Nothing is lost, and it works when the commit is someone else's.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
