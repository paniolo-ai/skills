---
source-slug: mcp-pooled-server
source-hash: d9196c94823863b630dd32a32f96e76b1e1bf11680786c354fdcb505e034b579
bundled: 2026-10-24
title: Pooled MCP Server for Multiple Agent Hosts
type: concept
tags:
- mcp
- agents
- server-architecture
- concurrency
updated: 2026-10-24
---

# Pooled MCP Server for Multiple Agent Hosts

A pooled MCP server is one long-lived application process serving several agent hosts. *Pooled* describes application ownership and reuse, not a third MCP transport or a guarantee that tool calls execute in parallel. This pattern is useful when hosts need the same warm index, job registry, or retained results.

- [Separate the entrances from the owner](#separate-the-entrances-from-the-owner)
- [Define what clients share](#define-what-clients-share)
- [Bound concurrent work](#bound-concurrent-work)
- [Make lifecycle explicit](#make-lifecycle-explicit)
- [Test with real clients](#test-with-real-clients)

## Separate the entrances from the owner

```text
agent A → MCP stdio adapter ───┐
agent B → MCP stdio adapter ───┼→ one state-owning server
agent C → MCP Streamable HTTP ─┘
```

An MCP host sees a standard transport: stdio or Streamable HTTP. A command-based host can launch a thin stdio adapter, which forwards JSON-RPC messages to the pooled process through [native-local-ipc](./native-local-ipc.md). A URL-capable host can connect to a separately exposed HTTP endpoint on the same process. An adapter must not initialize its own full server or the state splits again.

The choice of transport does not decide whether state is shared. One HTTP process can own several clients; several stdio adapters can also share one process through local IPC. SSE streaming is optional for responses that need progress or multiple server messages. Ordinary JSON responses are sufficient for many calls.

## Define what clients share

List every object that belongs to the process: open indexes, caches, job handles, retained reports, and cancellation state. Give each request a client-local response path while the server owns shared work. Preserve JSON-RPC IDs in responses, but do not treat an ID as globally unique across hosts.

Decide whether all local hosts have the same authority. A shared local credential proves access to the pool; it does not identify a vendor or isolate its jobs. If reports or write actions must be private to one host, add application identities and authorization. Do not infer ownership from an HTTP connection or optional MCP session identifier.

Define functional parity for operations exposed through both the direct CLI and MCP. A shell command can become a thin pool client only after the server has an equivalent operation contract, including validation, output, and write policy. Until then, document that the direct command uses a separate execution path.

## Bound concurrent work

Accept several clients while limiting open connections, message size, queued requests, and execution time. One dispatcher can serialize tools that use a non-thread-safe core; concurrency at the connection layer still lets several hosts stay attached. If tools run in parallel, protect shared state and test cross-client response routing.

Use backpressure when the queue is full. Report a retryable busy condition to the requesting client, rather than opening another full server or letting work grow without limit. Keep notifications, requests, and responses distinct: an MCP notification has no JSON-RPC response.

## Make lifecycle explicit

Use one owner per configured root, with an exclusive lock and a verified endpoint identity. An old socket file or port record does not prove a live server. State which command starts the pool, how adapters discover it, whether startup is automatic, and what happens after a crash.

Offer a compatibility path when native sockets or an HTTP-capable host are unavailable. Private stdio can keep a single host working, but it has a separate job registry. Make that mode explicit so an agent does not assume its jobs are visible to another host. Do not replay a write request automatically after a connection drops unless the operation has a sound idempotency contract.

## Test with real clients

Connect two independent MCP clients with overlapping requests and duplicate JSON-RPC IDs. Verify that each receives its own response and can see intentionally shared jobs. Disconnect one client during a long call; the other should remain usable. Exercise queue saturation, invalid credentials, wrong-root endpoints, stale endpoints, shutdown, and restart.

Run the connection tests on every supported operating system and with representative agent hosts. A protocol fixture proves routing, but it cannot prove that a host exposes the required command, URL, and credential settings.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
