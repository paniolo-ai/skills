---
source-slug: mcp-testing-evals
source-hash: 669fe63e00d02894c85acf6e2f2a5e9dd7b45b954692d71115d1b76d2b4ae976
bundled: 2026-10-24
title: MCP Testing and Evals
type: concept
tags:
- mcp
- testing
- evals
- conformance
updated: 2026-10-24
---

# MCP Testing and Evals

MCP servers have two distinct correctness surfaces, and they need different test machinery: **protocol conformance** (did we implement the spec?) and **agent behavior** (does the model pick the right tool?). A green typecheck says nothing about either.

## Layer 1 — protocol conformance

**MCP Inspector** (`@modelcontextprotocol/inspector`, v2) is the interactive floor — Web UI, scriptable `--cli` for CI and agent feedback loops, and a TUI. Use it to drive `initialize` → `tools/list` → `tools/call` by hand before trusting a client. Caveat: it stores OAuth tokens and `env:` secrets in the OS keychain or `~/.mcp-inspector/secrets.json` — treat it as a credential-bearing tool, not a harmless debug UI.

**The conformance suite** (`@modelcontextprotocol/conformance`) runs spec-pinned scenarios against clients *and* servers:

```bash
npx @modelcontextprotocol/conformance server --url http://localhost:3000/mcp
npx @modelcontextprotocol/conformance client --command "<cmd>" --suite core
```

- Suites: `core`, `extensions`, `backcompat`, `auth`, `metadata`, `draft`,
  plus SEP-specific suites — extension scenarios exist per-SEP (SEP-2640
  skills ships `sep-2640.yaml`).
- Every scenario also runs **wire-schema checks** — each JSON-RPC message
  validated against the spec's schema for the negotiated version.
- `--spec-version` filters by spec version; `--requirements <rev>` freezes the
  required set at a release so a growing suite doesn't move your target.
- The suite exercises both lifecycles: stateful (≤2025-11-25) and the draft's
  stateless per-request-`_meta` model — run both if you support both.

**In-repo contract tests** remain the cheapest gate: assert `tools/list`
output matches a snapshot, every `op` enum value is reachable, annotations are
honest, and no schema accepts forbidden params. `paniolo-mcp`'s
`tests/contract.rs` is the local pattern.

For a server shared by several MCP clients, add a separate integration gate:

1. Start one server and initialize two independent clients with different JSON-RPC IDs. Keep both connected and overlap a slow read with a second request. Assert each client receives only its own result.
2. Repeat with the actual vendor hosts and record their versions, transport configuration, and successful tool calls. A synthetic client cannot prove that a vendor can configure the required credential.
3. Try a missing or invalid credential, invalid `Host`/`Origin`, unsupported protocol version, and wrong configured root. None may dispatch a tool or attach to a different root's process.
4. Fill the request queue and verify the response is bounded and retryable. If long-running jobs are shared, test whether another client can inspect or cancel a handle according to the declared access policy.

Run transport tests against each protocol revision the server claims. A 2025
sessionful test does not establish 2026 draft compatibility, and a passing
`tools/list` test does not prove request isolation or useful concurrency.

## Layer 2 — agent behavior evals

Conformance proves the wire is right; it does not prove the model can find or
use your tools. That takes evals:

- **Build task evals, not vibes.** Anthropic's tool-design guidance: assemble
  realistic tasks, measure tool *selection* and call correctness, iterate on
  descriptions against observed failures. Description text is a routing
  prompt — tune it empirically.
- **Test the discovery path, not just the call path.** Does the model pick the
  tool when it should? Does it hallucinate a leaf command that doesn't exist
  in an op-enum design? These fail silently in a conformance-only world.
- **Measure token cost as a metric.** Result verbosity and catalog size are
  evaluable numbers, not aesthetics — see [mcp-token-efficiency](./mcp-token-efficiency.md).

## Ordering

1. Contract tests in CI on every tool change.
2. Inspector/CLI smoke when protocol surface changes (new methods, new
   capability declarations).
3. Conformance suite against a real build before claiming spec support —
   especially for extensions.
4. Multi-client integration and vendor-host smoke tests before claiming one server supports multiple agents.
5. Behavioral evals when descriptions or tool granularity change — the wire
   being right doesn't mean the agent can drive it.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
