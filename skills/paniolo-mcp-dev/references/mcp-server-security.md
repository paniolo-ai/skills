---
source-slug: mcp-server-security
source-hash: 39c770bea1104b644ba5afd832b533aaf709e2a2aa83653b6315e17b388fa911
bundled: 2026-09-30
title: MCP Server Security
type: concept
tags:
- mcp
- security
- annotations
- agents
- best-practices
updated: 2026-09-30
---

# MCP Server Security

The security floor for an MCP server is not the spec's annotations — it is
least-privilege scope, honest labeling, consent before mutation, and never
forwarding credentials. Tool design lives on [mcp-tool-design](./mcp-tool-design.md); context
budgets on [mcp-token-efficiency](./mcp-token-efficiency.md).

## Annotations are hints, not guarantees

`ToolAnnotations` carries four risk hints — `readOnlyHint`,
`destructiveHint`, `idempotentHint`, `openWorldHint` — plus `title`. Every one
is a **hint**: the spec requires clients to treat annotations from untrusted
servers as untrusted, and defaults are deliberately pessimistic (unannotated
reads as non-read-only, destructive, non-idempotent, open-world).

- **Set them honestly anyway.** `readOnlyHint: true` on reads,
  `destructiveHint: false` on additive writes, `openWorldHint: false` on
  closed-domain tools. Trusted hosts use them to skip or require confirmation.
- **They cannot stop a lying server or a prompt-injected model.** Enforcement
  lives in sandboxing, network controls, and scope — not a boolean on the
  schema.
- **Risk is session-level.** The "lethal trifecta" — private-data access +
  untrusted-content exposure + external communication — is a property of the
  whole tool mix in a session, not any single tool. `openWorldHint` is the
  hook a host can use to flag a trust-boundary crossing; treat every tool
  result that touches the outside world as untrusted content.

## Human in the loop

The spec says applications SHOULD show which tools are exposed, mark
invocations visibly, and present confirmation prompts — a human able to deny a
call is the baseline, not an optional feature. Map mutating tools to
`destructiveHint` / `readOnlyHint: false` so hosts know where the consent gate
belongs, and plan writes as plan-first-apply-second so the gate has something
concrete to approve.

## Never pass through tokens

- MCP servers **MUST NOT** accept tokens that were not issued to them, and
  **MUST NOT** forward client tokens to downstream APIs. Passthrough defeats
  audience binding, breaks accountability and audit trails, and turns your
  server into an exfiltration proxy for stolen tokens.
- Validate the audience on every presented token; issue and check your own.

## Confused deputy (proxies)

An MCP server proxying a third-party API with a **static client ID** plus
**dynamic client registration** can be tricked into delivering authorization
codes to an attacker — the third party's consent cookie skips the consent
screen. Required mitigations: per-client consent stored server-side before
forwarding, a consent page naming the client/scopes/redirect, exact-match
`redirect_uri` validation, and a cryptographically random `state` per request.

## Scope minimization

- Start with a minimal scope set (low-risk discovery/read); elevate via
  targeted `WWW-Authenticate` `scope="..."` challenges when a privileged
  operation is first attempted.
- Emit precise challenges rather than the full catalog; log elevation events;
  accept down-scoped tokens. Omnibus scopes widen blast radius, hide intent in
  audits, and push users to abandon consent dialogs.

## Transport and surroundings

- **stdio** limits access to just the MCP client — the right default for local
  servers. But a *proxy* that spawns stdio servers for web clients turns an
  XSS into system compromise; isolate it (container, restricted environment).
- Any HTTP transport needs an auth token or restricted IPC.
- Clients fetch server-supplied URLs during OAuth discovery — SSRF into
  internal networks and cloud metadata endpoints is a real vector; egress
  controls apply on both sides.
- Native clients using `localhost` redirect URIs can be impersonated;
  authorization servers are expected to warn and display the redirect host.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
