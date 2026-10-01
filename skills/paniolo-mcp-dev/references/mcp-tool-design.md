---
source-slug: mcp-tool-design
source-hash: f8e168858979b76147fb6d2f02e46e1c484702c173cdb878e0c4461bbeb80494
bundled: 2026-09-30
title: MCP Tool Design
type: concept
tags:
- mcp
- tools
- agents
- best-practices
updated: 2026-09-30
---

# MCP Tool Design

Tools are model-controlled: the language model discovers and invokes them from
context, so a tool is a contract between a deterministic system and a
non-deterministic caller. Design for the caller that occasionally hallucinates,
answers from general knowledge instead of calling, or picks the wrong tool —
not for a deterministic client. Response shaping and token budgets live on
[mcp-token-efficiency](./mcp-token-efficiency.md); risk labeling and consent on [mcp-server-security](./mcp-server-security.md).

## Choose tools by workflow, not API surface

- **Fewer, sharper tools beat catalogs.** Wrapping every endpoint or primitive
  wastes context and confuses selection. Target high-impact workflows that
  match real evaluation tasks, then grow.
- **Consolidate what agents chain.** `schedule_event` (find availability +
  create) beats `list_users` + `list_events` + `create_event`;
  `search_contacts` beats `list_contacts` + read-every-row; a compiled
  `get_customer_context` beats `get_customer_by_id` + `list_transactions` +
  `list_notes`. Every consolidation removes intermediate results from the
  model's context.
- **Never expose store internals** an agent should not orchestrate (locks,
  digests, raw objects). One clear job per tool; overlapping tools compete for
  selection.
- **Search beats enumerate.** Agents pay per token; a `search_logs` returning
  relevant lines plus context replaces a `read_logs` brute-force scan.

## Namespacing

- Group related tools under common prefixes — by service (`asana_search`,
  `jira_search`) or resource (`asana_projects_search`) — so tools stay legible
  beside other servers' tools in one session.
- Prefix- vs suffix-style schemes measurably differ per model; pick by your own
  evaluation, not habit.

## Schemas and descriptions

- **Write descriptions like onboarding a new hire.** Make implicit context
  explicit: query formats, niche terms, relationships between resources. Small
  description edits produced dramatic error-rate and completion gains on
  SWE-bench Verified.
- **Name inputs unambiguously** — `user_id`, not `user`; `start_date`, not
  `start`. Enforce shape with strict schemas rather than prose warnings.
- **Prefer `enum` values that explain themselves** and concrete types over
  free-form strings. Normalize in server code; never make the model do string
  transforms or mental math.
- Declare `inputSchema` for every tool; add `outputSchema` when results are
  structured so clients can validate `structuredContent`.

## Result and error channels

- Return `structuredContent` for machine-shaped results, with an equivalent
  text block for backward compatibility.
- Tool errors belong in the result (`isError: true`), not protocol errors —
  reserve JSON-RPC errors for unknown tools, bad arguments, and server faults.
- Prompt-engineer error text: say what was wrong and what a valid call looks
  like. An opaque code costs a retry loop; an actionable message costs one
  call.
- Results can carry `resource_link` or embedded resources — use them to point
  at bulk data instead of inlining it.

## Evaluate, then iterate

- Build evaluation tasks from real usage — multi-step, multi-tool, messy — and
  pair each with a verifiable outcome. Sandbox-trivial tasks overfit.
- Track per-task accuracy plus total tool calls, tokens, runtime, and errors.
  Redundant-call patterns mark candidates for consolidation.
- Read transcripts, including reasoning before tool calls — what agents omit
  from their feedback matters as much as what they say. Let agents critique
  and even rewrite their own tool descriptions, then re-measure.

## Protocol mechanics to honor

- Declare the `tools` capability; set `listChanged` when the set can change.
- `tools/list` paginates — large registries should page rather than dump.
- The spec says a human should be able to deny tool invocations; design tools
  assuming the host may gate each call.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
