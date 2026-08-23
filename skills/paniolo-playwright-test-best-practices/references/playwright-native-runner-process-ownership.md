---
source-slug: playwright-native-runner-process-ownership
source-hash: ae5fc2f7d0a4c5f9c2101c138e6dc21fbdc1fb6cb5854e12204ee8ffcc4d20dc
bundled: 2026-08-23
title: Native Runner Process Ownership
type: concept
tags:
- authoring
- playwright
- testing
- windows
- scripts
updated: 2026-08-20
---

# Native Runner Process Ownership

A local Playwright wrapper owns the exact server processes it starts. Portable wrappers should use
a typed TypeScript/Bun entrypoint, direct argv arrays, an explicit working directory, and an
explicit environment.

## Startup Contract

- Start a fresh stack when the command promises isolation.
- Refuse to kill an unowned listener on a required port; report it and stop.
- Write logs and a PID registry under the operating system's temporary directory.
- Record each direct child process ID immediately after spawning.
- Wait for explicit readiness markers or health checks before launching Playwright.
- After readiness, enumerate every registered root's descendants and persist those IDs too.
- Register directly spawned grandchildren when the wrapper can observe them.
- Surface early child exit with its exit status and relevant log tail.
- Use `process.execPath` or an explicit runtime path instead of assuming a shell runtime is on PATH.

Persisting descendants after readiness matters because an external timeout or supervisor can kill
and detach a root before final cleanup. Once the parent relationship disappears, a later cleanup
process may be unable to rediscover those descendants from the root ID alone.

**Observed 2026-08-21.** A `wrangler dev` was stopped by killing its shell wrapper. The node
supervisor underneath survived and kept **respawning** `workerd` children until the supervisor
process itself was killed. Killing the root is not the same as terminating the tree, and a
supervisor will actively rebuild what a partial kill removed — which is why the shutdown contract
re-enumerates and then verifies through an independent listener query rather than trusting the
signal. A suite that starts an API and a web app has exactly this shape, so a partially stopped run
leaves listeners holding the ports the next run needs.

## Shutdown Contract

1. Re-enumerate descendants while registered roots are still alive and update the registry.
1. Terminate registered process trees in dependency order.
1. Attempt cleanup of every persisted descendant even when its recorded root no longer exists.
1. Wait for termination and record failures instead of assuming a signal succeeded.
1. Check expected ports through an independent listener query.
1. Remove only owned PID and log metadata after the listener check completes.

A missing registry is not cleanup evidence. The completion record must include both the registry
state and an independent post-run listener result. If a listener remains, identify its PID and
ownership before taking further action.

Never kill every process with a broad executable name. Unrelated development sessions may use the
same runtime, command name, or port range.

## Argument Forwarding

Read the package script before invoking it. Some package managers forward arguments directly, so a
literal `--` can become a positional separator for Playwright and change project or file selection.
Place options where the script expects them, then verify the banner reports the intended project,
test count, and worker count.

## Verification

Test root and descendant registration, an already-dead root with a surviving recorded descendant,
an unowned listener, repeated cleanup, and signal handling. Run the real wrapper on every supported
host family because process-tree discovery and termination are host-sensitive.

## See Also

- Local Runner Wrappers
- Startup Failures
- TypeScript Bun Script Portability
