---
source-slug: mcp-transports-auth
source-hash: fc934845d5a8bcf1d73567bb5c021fd933a291ab8744967e1bf99990d3e68d7a
bundled: 2026-10-24
title: MCP Transports and Authorization
type: concept
tags:
- mcp
- transports
- security
- auth
updated: 2026-10-24
---

# MCP Transports and Authorization

MCP messages are JSON-RPC over two standard transports. Choose by who owns the process and how many clients need it. Streaming is a response option within HTTP, not the reason multiple clients can connect.

## The two transports

| | `stdio` | Streamable HTTP |
| --- | --- | --- |
| Shape | child process, stdin/stdout | HTTP POST + optional SSE stream, one endpoint |
| Lifecycle | client owns the process | server is a network service |
| Sessions | implicit in one process | optional `Mcp-Session-Id` in 2025 revisions; no protocol session in the 2026-07-28 draft |
| Auth | none in-band — inherits the user's environment | full HTTP auth surface |

- **stdio** is the default for one local host: no port or listener lifecycle, the host owns the process, and it dies with that host. It remains a current transport, not a deprecated predecessor.
- **Streamable HTTP** lets an independent server accept multiple client connections at one endpoint. A `POST` may return one JSON object or a request-scoped SSE stream. Choose SSE when the call needs progress or multiple server messages; ordinary JSON is enough for a simple tool response. An optional `GET` SSE stream belongs to the 2025 transport revisions; the 2026-07-28 draft removes it.
- **Shared local server** is its own deployment case: bind to loopback, authenticate each HTTP request, and verify that each vendor can supply the credential. A host without that ability can use a small stdio-to-HTTP adapter while still sharing one server process. Do not claim multi-vendor support from a two-client protocol test alone; exercise actual host configurations.

## The stateless turn (draft)

The 2026-07-28 draft reframes the protocol as **stateless**: every request carries its own `_meta` (protocol version, capabilities context), and no connection, process, or session ID implies conversation continuity. Anything spanning requests needs an explicit identifier (task handles, subscription IDs). Even on a 2025 implementation, do not mistake a transport session for authorization to read another client's job. Explicit handles need an access policy.

## Multi-client server contract

- **Negotiate only versions the server implements.** Never echo an arbitrary requested version. For a sessionless 2025 HTTP server that supports one version, validate each later `MCP-Protocol-Version` header against that supported version; do not store one negotiated version on the listener for all clients. If several versions are supported, retain genuine per-client negotiation state or use a per-request version contract appropriate to the protocol revision.
- **Separate accepting requests from executing tools.** Several clients can stay connected while tools execute serially. A bounded queue and one dispatcher are often a safe first cut for a server with non-thread-safe state. If work is parallelized, protect each shared resource and prove responses never cross clients.
- **Bound resources and retries.** Set request size, concurrent connection, queue, and execution limits. Return an actionable retry response when saturated. Long-running jobs need explicit IDs, idempotency behavior, cancellation, and result retention rules.
- **Identify the right server.** If one process is meant to serve one configured root, lock by canonical root and verify the root's identity before reusing a listener. A matching port or `serverInfo.name` does not prove that two processes serve the same data.
- **Treat all holders of one local token as one authority.** A shared token does not distinguish agents. If jobs or reports must be private per agent, add application identities and server-side authorization; do not key that policy on TCP connections or `Mcp-Session-Id`.

## Authorization by deployment

The MCP authorization framework applies to HTTP transports. stdio servers inherit the launching user's ambient authority and need no in-band auth. A local HTTP listener still creates a new access path, even when bound to loopback.

- **Remote or multi-user HTTP:** the server is an OAuth 2.1 resource server. Clients obtain tokens from the authorization server *upstream* of MCP; the MCP server validates them (audience-checked to itself via RFC 8707 resource indicators) — it does not see user credentials.
- **`401` + `WWW-Authenticate` is the entry point**; clients discover the auth server from resource metadata, then run a standard OAuth flow (ideally via the host's browser/elicitation URL mode, not inside model context).
- **Token passthrough is explicitly forbidden** — a server MUST NOT forward its inbound token to another API, and MUST NOT accept tokens it wasn't issued for. This is the anti-confused-deputy rule; see [mcp-server-security](./mcp-server-security.md).
- **Local HTTP:** bind to `127.0.0.1`, validate `Host` and present `Origin` headers, and require a local token or restricted IPC when tools can read private data or write files. Keep credential values out of tracked configs, URLs, and logs. A same-user malicious process may still be within the local trust boundary.

## Checklist

- One local host → stdio. Several local hosts sharing one process → authenticated loopback Streamable HTTP. Remote/multi-tenant → HTTPS Streamable HTTP with OAuth resource-server validation.
- For shared HTTP, test two independently initialized clients, overlapping calls, version headers, response routing, root identity, missing/invalid credentials, and queue saturation.
- Never accept tokens/credentials as tool arguments or per-call config.
- Don't bind session semantics to connection lifetime — the draft makes that illegal, and HTTP intermediaries make it unreliable regardless.
- For OAuth-protected remote servers, emit `WWW-Authenticate` on 401s and validate audience on every inbound token.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
