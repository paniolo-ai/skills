---
source-slug: native-local-ipc
source-hash: f09521457fe8e19e42afd0eb41e8c0839b234701adcfcaede3dceb87fcbdf116
bundled: 2026-10-24
title: Native Local IPC for Shared Agent Servers
type: concept
tags:
- mcp
- ipc
- transports
- agents
updated: 2026-10-24
---

# Native Local IPC for Shared Agent Servers

Native local interprocess communication (IPC) lets several processes talk to one service on the same machine without exposing the service through an HTTP port. For an agent server, a short-lived stdio adapter or CLI command can be a client of a long-lived local process that owns warm indexes, jobs, and reports.

- [The connection shape](#the-connection-shape)
- [Cross-platform choices](#cross-platform-choices)
- [Server contract](#server-contract)
- [Testing the claim](#testing-the-claim)
- [Paniolo application](#paniolo-application)

## The connection shape

```text
agent host → stdio adapter ─┐
CLI command → local client ─┼→ local IPC socket → one state-owning process
another host → adapter ────┘
```

The host still sees the transport it supports. An MCP host with only command configuration launches an adapter and speaks standard MCP stdio to it. The adapter forwards messages over IPC. Shell commands can use the same service once their operations have a server API. A direct HTTP endpoint may be offered separately for hosts that support URL-based MCP; it is not required for the local pool.

The pool should own state that must be shared: open indexes, job handles, retained results, and cancellation. Client processes should own only their input/output streams, request correlation, and reconnection behavior. Starting one full server per adapter would recreate the original state split.

## Cross-platform choices

On macOS and Linux, Rust provides `UnixListener` and `UnixStream` in [`std::os::unix::net`](https://doc.rust-lang.org/std/os/unix/net/). The ordinary local process path is the same on both. A macOS app sandbox changes the access question: [Apple's App Groups guidance](https://developer.apple.com/documentation/bundleresources/entitlements/com.apple.security.application-groups) says sockets used between sandboxed apps, or between a sandboxed app and a nonsandboxed app, belong in a shared app group container. Test how the actual agent host launches its adapter before assuming it can reach a socket in the user's local state directory.

Windows has AF_UNIX support, and the [`uds_windows` crate](https://crates.io/crates/uds_windows) provides analogous listener and stream types. Its README names Windows 10 version 1809 and Windows Server 1809/2019 as generally available support. Test the exact Windows versions the product supports before treating one socket implementation as portable.

Keep the socket path short enough for the platform and place it under user-scoped local state. On Unix, restrict the directory and socket permissions. On Windows, verify the directory's access control and the socket behavior on supported versions. A local socket is an access boundary, but it is not an agent identity: several adapters running as one user may still share authority.

An authenticated loopback TCP connection is another local IPC option when native sockets are unavailable. It needs a port, host/origin handling if it carries HTTP, and a local credential. Choose it for an explicit compatibility reason. A private stdio server is a final compatibility path when the shared service cannot run; it works for one host but does not share jobs or warm state with other hosts.

Offer these as documented connection modes so agents can choose a path that their host and operating system support. Do not silently fall back from a shared pool to private stdio: that changes job visibility and may make the same command behave differently across agents. A failed shared connection should report which alternatives are available and how to select one.

## Server contract

- **Frame messages.** Use a length prefix or a documented line-delimited JSON format so one read cannot be mistaken for one complete message. Preserve JSON-RPC request IDs and send each response to the client that made the request.
- **Authenticate before dispatch.** A user-scoped socket path or shared token controls local access. A shared token grants shared local authority; it does not isolate jobs per vendor or agent.
- **Bound waiting.** Limit message size, connected clients, queued calls, and per-call time. Keep accepting or rejecting connections while a slow tool runs. One dispatcher may serialize tool execution without forcing clients to connect one at a time.
- **Own lifecycle once.** Use an exclusive per-root lock or equivalent ownership record. An adapter that finds an existing pool must verify its root identity before reusing it; a stale socket file alone proves nothing.
- **Make restart visible.** Define whether clients reconnect, fail clearly, or start the pool when it is absent. A write request must not be silently replayed after a connection loss.

## Testing the claim

Start one service, connect two independent adapter processes, and overlap requests with distinct IDs. Verify response routing, shared job visibility, queue saturation, and that only one warm core was opened. Test a different root, a stale socket path, a missing credential, and a client that disconnects while a call runs. Repeat the connection test on every supported operating system; a synthetic socket test cannot prove a real agent host launches the adapter correctly.

## Paniolo application

Paniolo's proposed shared MCP process follows this pattern: command-based MCP hosts keep stdio, thin adapters reach one local pool, and direct HTTP is an additional entrance for hosts that support it. CLI families can move onto that pool as their MCP/CLI operation contracts reach functional parity. The Paniolo-specific root policy, command names, and migration gates belong in its product design rather than in this general pattern.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
