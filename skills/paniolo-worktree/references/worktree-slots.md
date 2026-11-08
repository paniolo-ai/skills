---
source-slug: worktree-slots
source-hash: 2580c82e985ea64d180a9e23b0681be9ec6fc5498f1de3d537554a5a41346532
bundled: 2026-11-07
title: Worktree Slots
type: concept
tags:
- git
- worktrees
- worktree-pool
- agents
- harness
updated: 2026-11-07
---

# Worktree Slots

A worktree pool gives one repository a fixed set of pre-warmed Git worktrees,
called slots, and hands each coding agent its own. It ships in
`@paniolo/cli` as `paniolo worktree pool`, and all of it described here is in
0.5.100 and later. This page is the model and the command contract; the
paniolo-worktree skill holds the rules an agent follows.

## Why slots

Agents that create their own worktrees put them anywhere, leave them behind,
and each one starts cold: a fresh dependency install and an empty build cache.
A pool fixes the count and the paths, so disk use is bounded and findable, and
a reused slot keeps its dependencies and caches warm for the next task.

## The model

- **Pool.** Declared under a repository in the harness `paniolo.config.json`
  (`repos.<key>.worktreePool`), so the product repo carries no agent files.
- **Slots.** Linked worktrees at fixed paths under the pool's `root`, in
  allocation order, each Git-locked so `git worktree prune` cannot remove it.
- **Owner.** Any nonempty string naming one agent session, such as its
  session ID. Every command that changes a claim takes `--owner` and refuses
  another owner's slot.
- **State.** One JSON file in the repository's Git common directory
  (`.git/paniolo/worktree-pool/state.json`), changed only under a lock. Never
  edit it by hand; every recovery has a command.
- **Primary checkout.** The repository's main checkout. It is not a slot, but
  it counts as slot `0` for port numbering and can carry an advisory lease.

| State | Meaning |
| --- | --- |
| `idle` | Clean, detached at the base ref, locked, free to claim. |
| `warming` | Reserved by an operation in progress. |
| `claimed` | Held by an owner on a topic branch. |
| `failed` | An operation stopped partway; files and any branch are kept, and the slot is unavailable until recovered. |
| `unknown` | Shown by `status` only: no state record yet. Use `init` or `adopt`. |

## Configuration

```json
"repos": {
  "my-app": {
    "path": "repos/my-app",
    "worktreePool": {
      "root": "repos",
      "slots": [".dev-worker-my-app-01", ".dev-worker-my-app-02"],
      "baseRef": "origin/main",
      "hostOs": "windows",
      "minFreeBytes": 1073741824,
      "preparation": [
        { "name": "pnpm", "program": "pnpm.cmd", "args": ["install", "--frozen-lockfile"],
          "timeoutSeconds": 900, "inputs": ["pnpm-lock.yaml"], "requiredOutputs": ["node_modules/.modules.yaml"] }
      ],
      "slotPorts": { "vite": 5100, "e2e-vite": 5200 }
    }
  }
}
```

- `slots`: 1 to 16 paths relative to `root`. Name them with a trailing number.
- `hostOs`: commands refuse to run on another OS, so a Windows slot is never
  driven from WSL and its caches stay valid.
- `preparation`: trusted steps run without a shell. A step reruns only when
  the fingerprint of its inputs and program versions changes, or a required
  output is missing.
- `cleanup`: fixed, bounded implementations (`cargo-stale-link-outputs`,
  `merged-branches`). They never force-delete and never touch a dirty tree.
- `slotPorts`: checkout `N` serves each name on `base + N`. A slot's number is
  its directory name's trailing digits; the primary checkout is `0`. Give
  end-to-end tests their own names, or a test run stops the dev server a
  person has open in that slot.
- A `paniolo.config.local.json` beside the config overrides the pool per
  machine (objects merge, arrays replace), so a laptop can run fewer slots.

## Commands

All take `--repo KEY` and print JSON. A refusal exits nonzero without changing
state, unless it reports that a slot was marked `failed`.

| Command | What it does |
| --- | --- |
| `status [--stale-hours N]` | Every slot's state, owner, branch, cleanliness, `unavailableReason`, claim age, last activity, and ports. Changes nothing. `--stale-hours` lists likely abandoned claims; it never frees them. |
| `init [--apply]` | Previews, then creates, every configured slot. Safe to rerun. |
| `adopt --path P [--apply]` | Imports an existing clean, detached, locked worktree at a configured path, keeping its caches. |
| `claim --owner O --branch B` | Reserves the first idle slot, fetches, detaches at the base, runs cleanup and changed preparation steps, creates the branch, and returns the path. Reuses this owner's existing claim. |
| `claim --owner O --from origin/B` | The same, starting from an existing remote branch such as an open pull request's. |
| `release --path P --owner O` | Returns the slot to idle, keeping caches, only if the tree is clean and the branch is merged, pushed under its own name, or equal to its upstream. |
| `refresh [--slot N] [--retry-failed]` | Moves idle slots to the latest base and reprepares them. Never touches a claimed slot. |
| `cleanup [--slot N] [--apply]` | Previews, then applies, the configured cleanup to reclaim space. |
| `env [--path P]` | Prints `PANIOLO_SLOT`, `PANIOLO_SLOT_PATH`, and `PANIOLO_PORT_<NAME>` for the checkout containing a path. |
| `prompt-status`, `hook` | A short ownership notice for agent hooks. |

`--format json` is accepted by `status`, `cleanup`, `claim`, `release`, and
`refresh`. Agents connected to the Paniolo MCP server use the `worktree` tool
(`status`, `env`) and the `worktree_write` tool (the writing commands, with
camelCase arguments). A claim whose preparation outlasts a client timeout can
go through `job_submit` as `worktree.claim`.

## Recovery

| Situation | Recovery |
| --- | --- |
| Slot `failed` during `refresh` | `refresh --slot N --retry-failed` |
| Slot `failed` during `cleanup` | `cleanup --slot N --apply --retry-failed` |
| Claim failed before its branch existed | The owner claims again or releases; anyone can `refresh --slot N --retry-failed`. |
| Claim failed after its branch existed | The owner saves the work, then releases. |
| A file held open (Windows) | Close the holder named in the diagnostic, then run its `nextAction`. The pool never kills a process. |
| An abandoned claim | Find it with `status --stale-hours 24`, confirm nothing runs there, push its work to a branch, then release it as its owner. |
| "Dubious ownership" | The checkout belongs to another OS account; transfer the folder before adopting it. |

Every retry first checks that the checkout is clean, detached, locked, and
linked, so no recovery can discard work.

## Harness repositories take no slots

A harness that contains its satellite repositories (under `repos/`, for
example) is the one repository that should not have a pool, and should not
get worktrees at all. Harness changes are validated against the satellites,
and a harness worktree has an empty `repos/`, so every config-driven gate in
it checks nothing real. The harness's only checkout is its primary,
slot `00`: agents take turns there on topic branches from `main` and hand it
back on `main` with no tracked changes. The optional primary lease
(`primaryLease: true`, `claim --slot primary`) records who holds it.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
