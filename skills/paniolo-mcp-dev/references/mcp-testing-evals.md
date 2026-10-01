---
source-slug: mcp-testing-evals
source-hash: f096b0c47f3ac59e21f6106aa5e7c2a616ccf742def87ae1d7adbe01e71143f2
bundled: 2026-09-30
title: MCP Testing and Evals
type: concept
tags:
- mcp
- testing
- evals
- conformance
updated: 2026-09-30
---

# MCP Testing and Evals

MCP servers have two distinct correctness surfaces, and they need different
test machinery: **protocol conformance** (did we implement the spec?) and
**agent behavior** (does the model pick the right tool?). A green typecheck
says nothing about either.

## Layer 1 — protocol conformance

**MCP Inspector** (`@modelcontextprotocol/inspector`, v2) is the interactive
floor — Web UI, scriptable `--cli` for CI and agent feedback loops, and a TUI.
Use it to drive `initialize` → `tools/list` → `tools/call` by hand before
trusting a client. Caveat: it stores OAuth tokens and `env:` secrets in the OS
keychain or `~/.mcp-inspector/secrets.json` — treat it as a credential-bearing
tool, not a harmless debug UI.

**The conformance suite** (`@modelcontextprotocol/conformance`) runs
spec-pinned scenarios against clients *and* servers:

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
4. Behavioral evals when descriptions or tool granularity change — the wire
   being right doesn't mean the agent can drive it.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
