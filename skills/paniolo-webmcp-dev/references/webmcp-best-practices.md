---
source-slug: webmcp-best-practices
source-hash: e5a016967b5d121b6320724eddf6321e1703ebca04455a13ff5c1436f743075a
bundled: 2026-09-25
title: WebMCP Best Practices
type: concept
tags:
- webmcp
- best-practices
- tools
- agents
updated: 2026-09-25
---

# WebMCP Best Practices

Tool declarations should be clear enough that agents succeed without probing
outputs and retrying. Apply these whether you use the imperative or declarative
API. Security annotations and origin gating live on [webmcp-security](./webmcp-security.md); API
mechanics on [webmcp-apis](./webmcp-apis.md).

## Tool strategy

- **One job per tool.** Avoid overlapping tools that compete for selection. Ask
  whether one function covers multiple tasks.
- **Register for page state.** Expose tools only while the UI can perform them;
  unregister with `AbortSignal` (or remove form attributes). Simple apps: static
  registration on load is fine. Complex multi-state UIs: dynamic lifecycle.
- **Stay under the context budget.** Every name, description, and schema costs
  tokens and latency. Lighthouse warns above **40** tools. Prefer a small set of
  sharp tools over a catalog.
- **Trust the agent.** Describe what the tool does and when to use it — not a
  rigid multi-step script in the description text.
- **Wrap existing app code.** `execute` should call the same path the UI uses
  (blog-damiangalarza-webmcp-without-mcp-server). Feature-detect
  `document.modelContext`, and poll briefly rather than checking once — see
  [webmcp-implementation-gotchas](./webmcp-implementation-gotchas.md) for why a single synchronous check can
  silently miss it.

## Names and descriptions

- Distinguish **immediate execution** from **starting a flow**
  (`create-event` vs `start-event-creation`).
- State what the tool does and when to use it. Prefer positive framing over
  "don't use this for…" constraints — limitations should fall out of a good
  description.
- Keep budgets short (Chrome guidance): ~30 chars for names, ~500 for tool
  descriptions, ~150 for parameter descriptions.

## Schemas that minimize model work

- Accept raw user input; normalize in your code (no mental math or string
  transforms required of the model).
- Use concrete types (`string`, `number`, `enum`) with parameter `description`
  fields.
- Prefer self-explanatory enum values (`"express"`) over opaque IDs (`1`).
- Explain *why* a choice exists when it is not obvious from the name.

## Reliability

- **Validate strictly in code, loosely in schema.** Schema constraints are
  hints; return actionable errors from `execute` so the agent can self-correct.
- Handle rate limits and failures with clear messages, or tell the agent to hand
  the task back to the user in the UI.
- **The browser does not sync the UI for you — `execute` must.** Nothing in
  WebMCP repaints the page when a tool runs; if `execute` mutates state without
  also updating the DOM the human sees, the human and the agent silently drift
  out of sync. Update visible UI state yourself when a tool finishes — humans
  and agents share the same screen.
- Pass `signal` into long work (`fetch`, etc.) so cancel stops real work
  ([webmcp-apis](./webmcp-apis.md)).

## Security checklist (pointer)

Set `readOnlyHint`, `untrustedContentHint`, and `consequentialHint` honestly;
keep auth/ownership/quotas on the server; never treat `exposedTo` casually —
details on [webmcp-security](./webmcp-security.md).

## Eval and debug

- Treat each tool like an API contract: inputs, outputs, constraints.
- Define baseline vs ideal results; evaluate with code checks and LLM-as-a-judge
  where quality is subjective.
- Prefer abstracting the tool (optional fields, ask the user) over narrow
  prompt patches for one model's quirks.
- Use Chrome's Model Context inspector / CDP and Lighthouse's registered-tools
  audit while iterating.
