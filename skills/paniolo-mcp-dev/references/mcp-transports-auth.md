---
source-slug: mcp-transports-auth
source-hash: 0d30c36030345f2868adedb2eea7eddf1b645f916b48d94f4727890995702ad6
bundled: 2026-09-30
title: MCP Transports and Authorization
type: concept
tags:
- mcp
- transports
- security
- auth
updated: 2026-09-30
---

# MCP Transports and Authorization

MCP messages are JSON-RPC over one of two transports. The choice is a
deployment decision first and a security decision second — everything about
sessions and auth hangs off it.

## The two transports

| | `stdio` | Streamable HTTP |
| --- | --- | --- |
| Shape | child process, stdin/stdout | HTTP POST + optional SSE stream, one endpoint |
| Lifecycle | client owns the process | server is a network service |
| Sessions | none beyond the process | `Mcp-Session-Id` header; resumable streams via event IDs / `Last-Event-ID` |
| Auth | none in-band — inherits the user's environment | full HTTP auth surface |

- **stdio** is the right default for local tool servers: no ports, no TLS, no
  auth machinery, and the process dies with the host. `paniolo mcp` is stdio.
- **Streamable HTTP** (the current HTTP transport; SSE-only transport is
  deprecated) exists for remote/multi-user servers. A single endpoint serves
  POSTs and can upgrade to an SSE stream; `Mcp-Session-Id` carries the
  negotiated session, and event IDs let a dropped stream resume without
  re-receiving.

## The stateless turn (draft)

The 2026-07-28 draft reframes the protocol as **stateless**: every request
carries its own `_meta` (protocol version, capabilities context), and no
connection, process, or session ID implies conversation continuity. Servers
MUST NOT infer context from prior requests on the same transport; anything
spanning requests needs an explicit identifier (task handles, subscription
IDs). Practical consequence: don't key server state on connection identity —
it will break under the draft lifecycle and already breaks under proxies and
retries today.

## Authorization: HTTP only, OAuth-shaped

The spec's authorization framework applies to HTTP transports — stdio servers
inherit the launching user's ambient authority and need no in-band auth (which
is also why a stdio server must not accept caller-supplied credential paths).

- **The server is an OAuth 2.1 resource server.** Clients obtain tokens from
  the authorization server *upstream* of MCP; the MCP server validates them
  (audience-checked to itself via RFC 8707 resource indicators) — it does not
  see user credentials.
- **`401` + `WWW-Authenticate` is the entry point**; clients discover the auth
  server from resource metadata, then run a standard OAuth flow (ideally via
  the host's browser/elicitation URL mode, not inside model context).
- **Token passthrough is explicitly forbidden** — a server MUST NOT forward
  its inbound token to another API, and MUST NOT accept tokens it wasn't
  issued for. This is the anti-confused-deputy rule; see
  [mcp-server-security](./mcp-server-security.md).
- **Local HTTP servers still need care**: validate `Origin` headers against
  DNS-rebinding, bind to `127.0.0.1`, and don't assume "localhost" means safe.

## Checklist

- New local server → stdio; new remote/multi-tenant server → Streamable HTTP
  + OAuth resource-server validation.
- Never accept tokens/credentials as tool arguments or per-call config.
- Don't bind session semantics to connection lifetime — the draft makes that
  illegal, and HTTP intermediaries make it unreliable regardless.
- Emit `WWW-Authenticate` on 401s; validate audience on every inbound token.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
