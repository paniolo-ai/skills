---
name: paniolo-queue
description: |
  Inspect and run commands through Paniolo host command queues (`paniolo queue` and the `queue` / `queue_write` MCP tools) to serialize heavy builds, test suites, and push gates across worktree slots and agents on one machine. Use when checking whether a host build lane is busy, queuing a heavy command in FIFO order, cancelling a waiting queue request, or reconciling an enrolled host queue. Do NOT use for claiming Git worktree slots (load `paniolo-worktree`) or for long-running MCP background jobs (`job_submit`).
license: MIT
metadata:
  version: 0.1.0
tags:
- queue
- concurrency
- worktrees
- cli
references:
- references/host-command-queues.md
---

**Requires:** terminal (`paniolo queue`) or Paniolo MCP tools (`queue`, `queue_write`).

# Host Command Queue Operations

Use `paniolo queue` (and the `queue` / `queue_write` MCP tools) to serialize heavy local commands across worktree slots, child subagents, hooks, and background workers on a single host. Read [host-command-queues](references/host-command-queues.md) for the full configuration schema, refusal codes, and broker recovery contract.

## 1. Inspecting Configured Lanes and Live Status

Before launching a heavy local build, test gate, or release command across multiple worktree slots, check which queues exist and whether the target lane is busy.

### Via MCP (`queue` read tool)

- List configured queues and provenance:
  ```json
  { "op": "list" }
  ```
- Check live active and waiting requests:
  ```json
  { "op": "status", "queue": "heavy-load" }
  ```

### Via CLI

```bash
paniolo queue list
paniolo queue status --queue <name>
paniolo queue config effective --json
```

- `list` and `config effective` read `paniolo.config.json` (plus any `paniolo.config.local.json` overlay) without starting the broker.
- `status` queries the live host broker. If the host is not enrolled or the broker is unavailable, `status` fails closed rather than assuming the lane is idle.

## 2. Running Commands Through a Queue Lane

Wrap heavy commands in `paniolo queue run` so concurrent worktrees wait in FIFO order instead of thrashing CPU, RAM, and disk:

```bash
paniolo queue run --queue <name> --label <op-label> --repository <repo-or-slot> -- <program> [args...]
```

- Always place `--` before the child program and its arguments; `run` executes the argument vector directly without shell interpolation.
- Pass clear `--label` (for example `pre-push-gate`) and `--repository` (for example `ranch-hand-slot-01`) strings so `paniolo queue status` shows human-readable owners.
- Use `--no-wait` (exits `75` if busy) when you want to probe a lane and immediately switch to bounded low-CPU work in a spare worktree slot instead of blocking.
- Use `--wait-timeout 5m` (exits `124` if not admitted in time) to bound queue wait time without imposing a runtime timeout once the child starts.
- With `--json`, queue outcome metadata is emitted on `stderr` so child `stdout` remains uncorrupted.

## 3. Coordinating Queues With Worktree Slots (N Slots, 1 Agent)

When one coordinating agent drives two or more warm worktree slots:

1. If a queue lane is occupied by Slot 01's push gate or build, either switch to Slot 02 for low-CPU preparation (edits, docs, focused checks) or submit Slot 02's heavy command through `paniolo queue run --queue <name>` so it starts automatically after Slot 01 finishes.
2. When delegating mechanical work on Slot 02 to a child subagent, instruct the child subagent either to stop before the heavy push gate or to run its gate through `paniolo queue run --queue <name>` so it cannot collide with Slot 01.

## 4. Cancelling a Waiting Request and Host Recovery

- **Cancel a waiting request:** Only requests still in `queued` state can be cancelled using their local request ID:
  - CLI: `paniolo queue cancel <request-id>`
  - MCP `queue_write`: `{"op": "cancel", "request": "<request-id>"}`
- **Enroll a host (one-time per Windows machine):**
  ```bash
  paniolo queue host enroll
  ```
- **Reconcile abandoned ownership (`OwnershipUnresolved`, exit `73`):** If a host crash left an unclosed broker journal and no live broker is listening, reconcile explicitly:
  ```bash
  paniolo queue host recover
  ```

## Do Not

- Do not attempt to invoke `run` through the `queue` or `queue_write` MCP tools; arbitrary command execution is CLI-only (`paniolo queue run -- <argv>`).
- Do not treat a `CoordinatorUnavailable` (`69`) or `OwnershipUnresolved` (`73`) error as "queue is idle"; resolve enrollment or recovery first.
- Do not pass `--wait-timeout` expecting it to kill a hung child process; `--wait-timeout` only bounds time spent waiting for admission.
- Do not run `paniolo queue host recover` while a live broker is healthy (it will refuse with `OwnershipUnresolved`).
- Do not bypass a configured host queue when another worktree slot on the same machine has an active heavy build or gate.

## References

- [host-command-queues](references/host-command-queues.md) — queue catalog config, CLI/MCP contract, refusal exit codes, and recovery
- [paniolo-worktree](../paniolo-worktree/SKILL.md) — worktree pool slots and N-slot single-agent coordination
- [paniolo-config](../paniolo-config/SKILL.md) — tracked vs `paniolo.config.local.json` overlay provenance

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
