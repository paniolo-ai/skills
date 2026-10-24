---
name: paniolo-mcp-dev
description: |
  Build, extend, debug, or review an MCP (Model Context Protocol) server. Use for transport and authentication choices, multi-client lifecycle, tool/resource design, output efficiency, long-running jobs, interoperability tests, or serving Agent Skills via SEP-2640. Applies to `paniolo mcp` and other MCP servers. Do not use for WebMCP page tools via document.modelContext; use paniolo-webmcp-dev.
license: MIT
metadata:
  version: 0.2.1
tags:
- mcp
- agents
- tools
references:
- references/mcp-beyond-tools.md
- references/mcp-discovery.md
- references/mcp-pooled-server.md
- references/mcp-server-security.md
- references/mcp-skills-extension.md
- references/mcp-testing-evals.md
- references/mcp-token-efficiency.md
- references/mcp-tool-design.md
- references/mcp-transports-auth.md
- references/native-local-ipc.md
---

**Requires:** file-read, file-edit, terminal (run the server, drive a client, run contract tests).

**Full reference:** [mcp-tool-design](references/mcp-tool-design.md) · [mcp-token-efficiency](references/mcp-token-efficiency.md) · [mcp-server-security](references/mcp-server-security.md) · [mcp-skills-extension](references/mcp-skills-extension.md) · [mcp-discovery](references/mcp-discovery.md) · [mcp-beyond-tools](references/mcp-beyond-tools.md) · [mcp-transports-auth](references/mcp-transports-auth.md) · [native-local-ipc](references/native-local-ipc.md) · [mcp-pooled-server](references/mcp-pooled-server.md) · [mcp-testing-evals](references/mcp-testing-evals.md)

Tools are model-controlled: the model discovers and invokes them from context, so every tool is a contract between a deterministic system and a non-deterministic caller. Two token costs dominate: **standing schema cost** (every advertised `inputSchema` sits in context every turn) and **intermediate results routed through the model** (each call's output re-enters context). Design on both axes. [mcp-token-efficiency](references/mcp-token-efficiency.md)

## Preconditions

- Know which hosts will connect, their supported protocol versions and transports, and which capabilities they implement. `resources`, `elicitation`, `sampling`, and the skills extension are unevenly supported. Test the intended hosts before claiming portability. [mcp-discovery](references/mcp-discovery.md)
- Decide whether the server is private to one host, shared among local hosts, or remote/multi-user. That decides process ownership, authentication, concurrency, and deployment checks. For a shared local process, define its state owner before choosing native IPC, HTTP, or both client entrances. [mcp-pooled-server](references/mcp-pooled-server.md)
- Pick the primitive by control flow before reaching for a tool: bulk/readable data → resources, canned user workflows → prompts, host-model work → sampling, user input mid-call → elicitation, long-running → tasks. [mcp-beyond-tools](references/mcp-beyond-tools.md)
- For `paniolo mcp` work, read the parity design first (`design-paniolo-mcp-cli-parity` in paniolo-wiki): the surface is family tools with `op` enums, not one tool per CLI command.
- Establish whether the operation is read-only, a plan→apply write, or long-running *before* choosing where it lands — that classification determines the tool it joins and the annotations it must carry.

## Defaults (proceed without asking)

- Add an op to an existing family tool before creating a new top-level tool.
- Read ops and write ops go in **separate tools** so `readOnlyHint` / `destructiveHint` stay honest.
- Results are concise-by-default with an opt-in detailed format; bulk bodies become `resource_link`s, never inline payloads.
- Long-running or agent-spawning work goes behind a submit/status/result job surface — never a synchronous tool that holds the request open.
- Check the user's authorization and the server-side policy before exposing credentials, process launch, or caller-supplied roots/config. Ask only when the authority or intended scope is genuinely missing; tool annotations alone do not grant it.

## Key rules

### Tool surface

- **Design from workflows, not your CLI/API tree.** Wrapping every leaf command produces an agent that can't pick a tool. Consolidate to the verbs the agent actually chains; ~40 leaf tools ≈ 15–25k tokens of permanent schema tax before the first call. [mcp-tool-design](references/mcp-tool-design.md)
- **An `op` enum needs per-op arg shapes** (`oneOf` branches on the input schema). A flat `args` bag invites guessing, and every invalid call is a paid round-trip.
- **Descriptions are routing prompts for the model.** Write when-to-use and when-not-to-use against observed selection errors, not prose summaries of the implementation. Iterate with evals, not intuition.
- **Name for a shared namespace.** The model sees your tools alongside every other connected server's — a generic `query` or `list` collides; a `paniolo_`-style or family-qualified name does not.

### Output and context cost

- **`response_format: concise|detailed`** on every read; concise returns top-N with semantic IDs, counts, and links to the full body.
- **Paginate with cursors** and make truncation steer the agent ("narrow with `filter`"), not just stop.
- **Semantic IDs over opaque IDs** in results — they measurably reduce hallucination; let callers opt into internal IDs when they need them.
- **Do not double-emit.** Returning identical JSON as both `structuredContent` and a text content block pays output tokens twice; the text fallback should be a one-line summary. [mcp-token-efficiency](references/mcp-token-efficiency.md)
- **Never make the model relay bulk data.** Fetch-then-update flows where the model copies a body into the next call's arguments are the canonical leak; code-execution hosts eliminate it entirely (Anthropic and Cloudflare both report ~98% reductions) — keep tool shapes compatible with that surface.

### Errors

- **Two channels, used correctly:** tool failures return `isError: true` in the tool result with an actionable `{code, message, remediation}` body the model can self-correct from; JSON-RPC errors are for protocol-level failures only (unknown method, malformed params).
- **Error text is agent-facing.** "what was wrong + what to try next" beats a stack trace or a bare exit-code echo.

### Security

- **Annotations are hints, not enforcement.** `readOnlyHint`/`destructiveHint` inform host consent UX; a hostile or buggy host ignores them. Real gates live server-side. [mcp-server-security](references/mcp-server-security.md)
- **Never pass tokens through a tool** — no credential passthrough, no session forwarding; confused-deputy and token-theft mitigations are the spec's security baseline.
- **Match the connection path to the host and OS**: private stdio for one local host; thin stdio adapters over native local IPC for a shared local pool; authenticated loopback Streamable HTTP for URL-capable local hosts; HTTPS Streamable HTTP with OAuth resource-server validation for remote users. Keep a documented fallback when native sockets or host HTTP settings are unavailable. Streaming responses are optional and do not create multi-client support by themselves. [native-local-ipc](references/native-local-ipc.md) [mcp-transports-auth](references/mcp-transports-auth.md)
- **Give a pooled server one state owner.** Keep the warm core, jobs, and reports in that process. Bound clients and queued work, route each response to its caller, and test two agent hosts with overlapping requests. [mcp-pooled-server](references/mcp-pooled-server.md)
- **Negotiate only implemented protocol versions.** Never echo an arbitrary client version or store one negotiated version for every HTTP client. Validate request metadata for the chosen version before dispatch. [mcp-transports-auth](references/mcp-transports-auth.md)
- **Separate transport sessions from application identity.** Jobs, reports, and other cross-call state need explicit handles and access checks. One connection, protocol session, or bearer token shared by several agents does not isolate their work. [mcp-transports-auth](references/mcp-transports-auth.md)
- **Write tools prove a plan first.** Default to showing the diff/blast radius and requiring `apply: true`; post-write, return a validation summary so the agent can confirm the world changed as intended.
- **Scope authority to config, not caller paths.** A tool that accepts a caller-supplied root or config is a path-traversal-shaped hole; address only configured targets.

### Serving skills (SEP-2640)

- Skills ride the `resources` primitive plus three methods: `skills/list` and `skills/get` (both mandatory once you declare `io.modelcontextprotocol/skills`) and optional `resources/directory/read`. The draft-era `skill://index.json` is gone — do not build it. [mcp-skills-extension](references/mcp-skills-extension.md)
- **Entries are complete manifests**: verbatim frontmatter + per-file `{uri, sha256 digest, size}` or `"dynamic"`. Serve static content with real digests so hosts can content-bind approvals; `"dynamic"` skills are unverifiable and hosts may refuse them.
- **Identity is (server identity, URI)** — `skill://a/SKILL.md` from two servers are unrelated skills; nothing you emit should encourage keying on URI or `name` alone.
- **Skill content is untrusted instructional text** once it leaves your server: hosts tag origin to the model, gate execution, and ignore `allowed-tools` on MCP-origin skills. Ship frontmatter that doesn't over-ask; within per-skill limits (512 files / 16 MiB).

## Output format

Code changes directly. After edits, state which tools/resources/methods were added or changed, which annotations were set and why, and how you verified the server advertises and answers them (initialize capabilities, tools/list, contract test). For reviews: cite the specific reference page for anything non-obvious.

## Error handling

- If a host can't see a tool, check `initialize` capability negotiation and `tools/list` before debugging the handler — a capability the host never declared silently receives no calls.
- If a write op appears uncallable, check the annotations/consent path — a `destructiveHint` tool under a strict host policy is gated, not broken.
- Never swallow a JSON-RPC error into a tool result, or a tool failure into a protocol error — each has a defined channel; crossing them breaks host retry and error UX.

## Validation

- For `paniolo mcp`: extend `crates/paniolo-mcp/tests/contract.rs` — op enum coverage, schema snapshots, annotation assertions, and the negative test that no schema accepts `config`/`root` params; then `bash scripts/check.sh` in ranch-hand.
- For any MCP server: drive `initialize` → `tools/list` → `tools/call` over every supported transport with a real client or a JSON-RPC script. Confirm negotiated capabilities and versions, advertised schemas, error channels, and authentication. For shared HTTP, use two independent clients with overlapping requests; check response routing, bounded waiting, and cross-client handle access. Use MCP Inspector and the spec-pinned conformance suite for protocol checks, then behavioral evals for tool choice. [mcp-testing-evals](references/mcp-testing-evals.md)

## Evaluations (I/O examples)

- "Add `stale list` to paniolo mcp" → add op `list` to the `stale` read tool with a `oneOf` branch, `readOnlyHint` already set on the tool, concise result with item IDs + `resource_link`s; not a new `stale_list` leaf tool.
- "Expose the full scan report" → resource `paniolo://scan/<target>/report` linked from the `scan` tool result, not an inlined findings array.
- "Ship the usage manual with the server" → `skill://paniolo/<name>/SKILL.md` resources + `skills/list`/`skills/get`, declared under `capabilities.extensions["io.modelcontextprotocol/skills"]`.
- "Let Codex and Cursor use one local server" → one authenticated loopback listener, two independently tested client configs, per-request version validation, and a concurrent-connection test; ordinary JSON responses suffice unless progress or subscriptions are needed.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
