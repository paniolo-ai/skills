---
source-slug: mcp-token-efficiency
source-hash: adb5f6714d7d52ca810763b9f0769302403fdbd5193c62a441848a81dea4d676
bundled: 2026-09-30
title: MCP Token Efficiency
type: concept
tags:
- mcp
- tokens
- context
- agents
- best-practices
updated: 2026-09-30
---

# MCP Token Efficiency

Two places tools spend the model's context: **definitions loaded upfront**
(every tool name, description, and schema sits in context before the first
call — thousands of tools means hundreds of thousands of tokens) and
**intermediate results routed through the model** (each call's output re-enters
context, and chained calls copy data through it again — a fetched document
passed to an update tool flows through twice). Design on both axes; tool count
and workflow shape live on [mcp-tool-design](./mcp-tool-design.md).

## Shape results for context, not completeness

- **Return high-signal fields.** Prefer `name`, `image_url`, `file_type` over
  `uuid`, `256px_image_url`, `mime_type`. Semantic identifiers measurably
  reduce hallucination; when a downstream call needs an opaque ID, let the
  caller opt into it.
- **Offer a `response_format` enum** (`concise` | `detailed`). Anthropic's
  worked example cut one Slack result from 206 to 72 tokens — roughly a third
  — by omitting IDs until they are needed.
- **Paginate, filter, and truncate by default.** Claude Code caps tool output
  at 25,000 tokens. A truncation notice should steer the agent toward narrower
  queries ("use `filter` or `page`"), not just stop.
- **Choose a format by measurement.** XML, JSON, and Markdown results score
  differently per model and task — there is no universal winner; evaluate.
- **Never make the model relay bulk data.** Fetch-then-update workflows where
  the model copies a document body into the next call's arguments are the
  canonical leak; see code execution below.

## Progressive disclosure for large registries

- Present tools as **files on a filesystem** or behind a `search_tools`
  meta-tool with a detail-level parameter (name only → name+description →
  full schema). The agent loads only what the current task needs.
- A flat catalog in `tools/list` is fine at tens of tools; at hundreds it is
  the dominant cost.

## Code execution: the ceiling on efficiency

Presenting MCP tools as a **code API** (TypeScript wrappers the agent calls
from a sandbox) instead of direct tool calls:

- **Cuts both costs at once** — definitions load on demand by reading files,
  and intermediate results stay in the execution environment. Anthropic's
  example dropped a workflow from 150,000 to 2,000 tokens (98.7%); Cloudflare's
  "Code Mode" reports the same finding independently.
- **Plays to model strength** — LLMs have seen vastly more real code than
  synthetic tool-call syntax, so they compose calls more reliably as code.
- **Moves control flow out of inference** — loops, polling, and conditionals
  run in the sandbox instead of alternating tool calls and sleeps through the
  model.
- **Enables privacy boundaries** — the client can tokenize PII between calls so
  sensitive values never enter context, and agents can persist intermediate
  state (or reusable helpers, packaged as skills) to files.
- **Has a real cost** — you are now running generated code: sandboxing,
  resource limits, and monitoring become your problem. Weigh it for bulk or
  chained workflows; simple read-only queries gain little.

## Ordering for a real server

1. Curate the tool list — consolidation is the cheapest token cut.
1. Make results concise-by-default with an opt-in detailed format.
1. Point at bulk data with resource links instead of inlining it.
1. For registry scale or multi-call workflows, add progressive disclosure or a
   code-execution surface.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
