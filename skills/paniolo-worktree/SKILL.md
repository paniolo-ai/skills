---
name: paniolo-worktree
description: |
  Rules for working in a Paniolo worktree pool: when to claim a slot, how to release one, what to do with busy, failed, or abandoned slots, per-slot ports, and why a harness repository keeps a single checkout. Use before starting work in any repository whose harness config declares a `worktreePool`, before creating a worktree of a harness repository, and when a slot is busy, failed, or looks abandoned. Do NOT use for general Git worktree mechanics in a repository with no pool; load `paniolo-worktree-best-practices` for that.
license: MIT
metadata:
  version: 0.3.0
tags:
- git
- worktrees
- worktree-pool
references:
- references/worktree-slots.md
---

## First: which kind of repository is this?

Check the harness `paniolo.config.json` for a `worktreePool` under the
repository's entry.

- **It has a pool:** work only in a claimed slot. Never `git worktree add`
  for it, and never edit its primary checkout.
- **It is the harness itself:** it gets no worktrees at all. Its primary
  checkout is the only one, slot `00`. See the harness section below.
- **It has neither:** load `paniolo-worktree-best-practices` for plain Git
  worktree discipline.

## Claiming

```bash
paniolo worktree pool status --repo <key>
paniolo worktree pool claim --repo <key> --owner <session-id> --branch <topic>
```

- Look before choosing: run `status` first.
- Use your session ID as the owner, the same one every time, so a repeated
  claim returns your existing slot.
- Work only in the path `claim` returns. Do not guess a slot from a directory
  listing.
- To continue an open pull request, claim with `--from origin/<branch>`.
- If no slot is idle, wait. A stale claim is still a claim.

## Releasing

Release only when the branch is pushed under its own name or merged, and the
tree is clean:

```bash
paniolo worktree pool release --repo <key> --path <slot> --owner <session-id>
```

`release` refuses anything else, and that refusal is protecting work. Push or
commit; never force it, reset the slot, or hand-edit pool state. Delete the
merged local branch afterward with `git branch -d`.

## Busy, failed, and abandoned slots

- **Owned by someone else:** never take over, reset, or release it.
- **`failed`:** the files and any branch are intact. Recover with the
  command the diagnostic names (`refresh` or `cleanup` with
  `--retry-failed`); every retry refuses a dirty checkout.
- **Looks abandoned:** `status --stale-hours 24` lists candidates; it never
  frees them. If the branch is already merged or pushed and the tree is clean,
  `reclaim --path <slot>` (0.5.101 and later) frees it: preview first, then
  add `--apply`. Anything else, `reclaim` refuses and names what would be
  lost; report it to a person, and push its work to a branch before anyone
  frees it.

## Ports

Never hard-code a slot's port or work it out from the directory name. Read it:

```bash
paniolo worktree pool env --repo <key>
```

It prints `PANIOLO_SLOT` and `PANIOLO_PORT_<NAME>` for the current checkout.
The primary checkout is slot `0`. Use the end-to-end port names for test runs,
so a test never stops a dev server someone has open in that slot.

## The harness keeps one checkout

A harness repository is validated against the satellite repositories it
contains, and a harness worktree has an empty `repos/`. So:

- Never create a worktree of the harness: no slot, no `repos/.pr-*`, no
  release or scratch checkout.
- Before editing, check the branch and `git status`. If another agent's
  branch or uncommitted change is there, stop and ask. Do not stash, reset,
  commit, or switch away from it.
- Branch from `main`, commit only your files, push, and open the pull request.
  Never leave a commit on local `main` unpushed.
- Before you finish, switch back to `main` with no tracked changes left.

## Gates check the configured checkout, not yours

Harness gates such as `paniolo wiki --config`, `paniolo scan --target`, and
`paniolo skills bundle --check` validate the repository path named in
`paniolo.config.json`, not your slot. So a pass may say nothing about your
files, and another session's unfinished work in that path can fail your
commit.

- Before trusting a pass, confirm the output names your checkout.
- To validate a slot, point a temporary config at it, run the gate, and
  delete the file:

  ```powershell
  $c = Get-Content paniolo.config.json -Raw | ConvertFrom-Json -Depth 60
  $c.repos.'<repo>'.path = '<path to your slot>'
  $c | ConvertTo-Json -Depth 60 | Set-Content paniolo.config.tmp.json
  paniolo wiki --config paniolo.config.tmp.json --fail-on error
  ```

- When a commit hook checks the wrong checkout, run the same check against
  yours, then commit with a temporary hook that runs that check. Never use
  `--no-verify` because someone else's work failed.

## Where worktrees must not go

- Never under `.cache/`. `pnpm install` runs `paniolo bootstrap`, which clears
  `.cache`, and takes the worktree and its uncommitted work with it.
- The `.dev-worker-` prefix belongs to pool slots. Do not give it to a
  worktree the pool does not manage. `status` lists any other linked worktree
  under the pool root as `unmanagedWorktrees`: report those, and check them
  for unpushed work before anyone removes them.

## Without a shell

Through the Paniolo MCP server, the `worktree` tool runs `status` and `env`,
and `worktree_write` runs `claim`, `release`, `refresh`, `cleanup`, `init`,
and `adopt` with the same rules and refusals. Submit a slow claim with
`job_submit` as `worktree.claim`.

## Do Not

- Do not create a worktree for a repository that has a pool, or for a
  harness repository.
- Do not take over, reset, or release another owner's slot. `reclaim` is the
  only way to free one, and only when nothing would be lost.
- Do not release a slot whose branch is not pushed or merged.
- Do not hand-edit `.git/paniolo/worktree-pool/state.json`.
- Do not trust a gate that validated the configured checkout as evidence about
  your slot.
- Do not put a worktree under `.cache/`.
- Do not drive a slot from another operating system than its `hostOs`.
- Do not delete slot build caches to free space; use `cleanup`, which keeps
  what makes the next build fast.
- Do not end a session with the harness checkout off `main`.

## References

- `references/worktree-slots.md` — the pool model, states, configuration,
  every command, recovery, and the harness rule.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
