---
source-slug: webmcp-evals
source-hash: 2855251fdd8ccd611adc92ced2aceb37b1f4a46cb7ad32af83964658df038176
bundled: 2026-09-25
title: WebMCP Evals
type: concept
tags:
- webmcp
- testing
- evals
- quality
updated: 2026-09-25
---

# WebMCP Evals

WebMCP tools sit next to an LLM. Unit tests catch JavaScript bugs; **evals**
catch probabilistic agent mistakes — wrong tool, wrong args, wrong order, or
acting on poisoned output. Keep both.

## What to prove

1. Descriptions/schemas make the tool's purpose clear.
2. Given intent, the model picks the right tool with correct parameters.
3. Prior tool output is used correctly on the next call.
4. End-to-end journeys succeed (and fail safely when they should).

Documented [webmcp-user-journeys](./webmcp-user-journeys.md) become the eval blueprint.

## Failure modes to cover

| Failure | Typical fix |
| --- | --- |
| Wrong tool / skips a step | Sharper description; less overlap; tool registered for current state |
| Wrong call order | Outputs that imply prerequisites; return "do X first" errors |
| Wrong arguments | Better `inputSchema` / enums / param descriptions |
| Bad or verbose tool output | Deterministic logic bugs; return only what the next step needs |
| Runtime / API errors | Graceful structured errors (retry vs fatal) |

Also eval **when not to call**: policy questions shouldn't hit catalog tools;
prompt injection in product text shouldn't become instructions.

## Test ladder

1. **Deterministic tool logic** — call the underlying functions (and
   `executeTool`) with valid/invalid/empty/cancelled cases. No model.
2. **Isolation evals** — user message → `expectedCall` (name + args). Include
   the **full** tool list for that app state so selection is realistic.
3. **Direct vs ambiguous prompts** — "add pepperoni" vs "all the meat toppings."
4. **End-to-end** — ordered steps plus unordered groups where order doesn't
   matter. Mid-chain: force state after a prior step and test the risky tool
   alone (e.g. discount before checkout).

Use paraphrases of the same goal — one passing phrasing proves little.

## Related

- [webmcp-best-practices](./webmcp-best-practices.md) — naming, schemas, reliability
- [webmcp-security](./webmcp-security.md) — injection via tool output
- [webmcp-observability](./webmcp-observability.md) — production signals that grow the eval set
