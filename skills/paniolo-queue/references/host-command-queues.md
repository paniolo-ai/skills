---
source-slug: host-command-queues
source-hash: 3c0c36464bfbd8fb9bde5a346c68f698e1204219354131c4ba2607027ca46849
bundled: 2026-11-20
title: Host Command Queues
type: concept
tags:
- queue
- concurrency
- worktrees
- agents
- harness
- cli
updated: 2026-11-20
---

# Host Command Queues

A host command queue (`paniolo queue`) serializes heavy local commands (builds,
test suites, pre-push gates, release cuts) across worktree slots, child
subagents, Git hooks, and background workers on one machine. Each configured
queue lane runs at most one command at a time (`maxConcurrent: 1`) in FIFO
order, supervised by an enrolled local broker so crashes and disconnects cannot
silently double-run a heavy gate.

This page is the model and the command contract; the paniolo-queue skill
holds the rules an agent follows.

## Contents

- [Why Host Queues](#why-host-queues)
- [Configuration and Provenance](#configuration-and-provenance)
- [Command Contract](#command-contract)
- [MCP Tool Contract](#mcp-tool-contract)
- [Refusal and Exit Codes](#refusal-and-exit-codes)
- [See Also](#see-also)

---

<a id="why-host-queues"></a>

## Why Host Queues

Worktree pools (worktree-slots) isolate Git working trees and per-slot
build caches (`CARGO_TARGET_DIR`), but every slot on a workstation still shares
one CPU, one SSD, and one pool of RAM. When two heavy gates overlap on the same
host, median gate duration roughly doubles and timeout or file-lock failures
climb.

Prompt rules alone cannot prevent overlap once a child subagent, a Git hook, a
background staleness worker, or a second terminal launches a build. `paniolo
queue` enforces single-lane admission at the OS level:

- **Explicit host enrollment.** A host is enrolled once with `paniolo queue host
  enroll`. Commands never invent an implicit fallback queue when enrollment or
  configuration is missing.
- **No shell interpolation.** `paniolo queue run --queue <name> -- <program>
  [args...]` executes the child argument vector directly after admission.
- **Fail-closed supervision.** If the connection drops while a job is running,
  the lane enters `reconciling` until the supervisor confirms quiescence or the
  operator runs `paniolo queue host recover`.

---

<a id="configuration-and-provenance"></a>

## Configuration and Provenance

Queues are declared under the top-level `queue` section of
`paniolo.config.json` (and optionally overridden in the gitignored
`paniolo.config.local.json` overlay):

```json
{
  "queue": {
    "queues": {
      "heavy-load": {
        "description": "Serialize heavy local Cargo builds and pre-push gates across slots",
        "scope": "host",
        "maxConcurrent": 1
      }
    },
    "operations": {
      "pre-push-gate": {
        "queue": "heavy-load"
      }
    },
    "agentStatus": {
      "enabled": true
    }
  }
}
```

- `queues.<name>`: 1 to 64 queues per catalog. Every queue currently requires
  `"scope": "host"` and `"maxConcurrent": 1`.
- `operations.<op>`: optional named operation mappings (`1..=128`) pointing to a
  configured queue name.
- `agentStatus.enabled`: whether agent status hooks surface queue activity.
- `paniolo queue config effective` (and `paniolo queue list --json`) report
  both the resolved catalog and per-pointer file provenance (`tracked` vs
  `local`).

---

<a id="command-contract"></a>

## Command Contract

All commands accept `--json`, `--config <path>`, and `--host-state <dir>`.

| Action | Command | Behavior |
| --- | --- | --- |
| List configured queues | `paniolo queue list [--json]` | Lists effective configured queues and descriptions without starting the broker. |
| Effective config & provenance | `paniolo queue config effective [--json]` | Prints queues, operation mappings, `agentStatus`, and `tracked`/`local` provenance. |
| Live queue status | `paniolo queue status [--queue <name>] [--json]` | Queries the live broker for active and waiting jobs on all queues or one named queue. Unavailable broker is reported as unavailable, never assumed idle. |
| Run through a queue | `paniolo queue run --queue <name> [--label <label>] [--repository <repo>] [--no-wait \| --wait-timeout <ms\|s\|m>] [--quiet] -- <program> [args...]` | Waits for FIFO admission on `<name>`, then runs `<program> [args...]`. With `--json`, lifecycle outcome JSON goes to `stderr` so child `stdout` stays clean. |
| Cancel waiting request | `paniolo queue cancel <request-id> [--json]` | Cancels a still-waiting (`queued`) request using its local receipt. Running jobs cannot be cancelled via `cancel`. |
| Enroll host | `paniolo queue host enroll [--json]` | Provisions a new private native host state directory and domain ID. |
| Recover host | `paniolo queue host recover [--json]` | Reconciles abandoned native jobs when the broker is not alive and restarts supervision. Refuses if the authority is already alive. |

### Waiting Flags on `run`

- Default: waits in FIFO order until admitted.
- `--no-wait`: refuses immediately with `Busy` (exit code `75`) if the queue is
  currently occupied.
- `--wait-timeout <duration>`: bounds how long the request may wait in line
  (for example `30s` or `5m`, refusing with `WaitTimeout` / exit code `124` if
  not admitted in time). It never kills a child after admission.

---

<a id="mcp-tool-contract"></a>

## MCP Tool Contract

The Paniolo MCP server delegates two tools to the `queue` component via
`--paniolo-tool-call`:

- **`queue` (read-only):**
  - `{"op": "list"}` — configured queue catalog and provenance.
  - `{"op": "config"}` — effective queue configuration and provenance.
  - `{"op": "status", "queue": "<optional-queue-name>"}` — live active and
    waiting requests from the host broker.
- **`queue_write` (mutating):**
  - `{"op": "cancel", "request": "<request-id>"}` — cancels a still-waiting
    request by receipt ID.

Arbitrary command execution (`paniolo queue run -- <argv>`) is intentionally
CLI-only and never exposed over MCP.

---

<a id="refusal-and-exit-codes"></a>

## Refusal and Exit Codes

When a child runs, `paniolo queue run` exits with the child's exit code. When
the queue refuses an invocation, it exits with a distinct sysexits-style code:

| Exit code | Refusal reasons | Meaning |
| --- | --- | --- |
| `69` | `CoordinatorUnavailable`, `UnqualifiedSupervisor`, `IncompatibleProtocol` | Host not enrolled, broker unreachable, or unsupported platform supervisor. |
| `73` | `OwnershipUnresolved` | Abandoned broker journal exists; inspect host and run `paniolo queue host recover`. |
| `75` | `Busy`, `QueueFull` | Queue occupied (`--no-wait`) or waiting capacity reached. |
| `77` | `PolicyConflict` | Registered queue policy conflicts with config. |
| `78` | `InvalidConfig`, `UnknownQueue`, `DuplicateRequest` | Missing/invalid queue config or unknown `--queue` name. |
| `124` | `WaitTimeout` | Request exceeded `--wait-timeout` before admission. |
| `126` | `LaunchFailed` | Admitted child executable failed to spawn. |
| `130` | `Cancelled` | Request was cancelled while waiting in line. |

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
