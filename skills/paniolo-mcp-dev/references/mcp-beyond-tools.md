---
source-slug: mcp-beyond-tools
source-hash: d9c99dfb225b8adb2cd086e8f8f6e292bb7c106c5a5554729f5ab62296a4826e
bundled: 2026-09-30
title: MCP Beyond Tools — resources, prompts, sampling, elicitation, tasks
type: concept
tags:
- mcp
- resources
- prompts
- sampling
- elicitation
updated: 2026-09-30
---

# MCP Beyond Tools — resources, prompts, sampling, elicitation, tasks

Tools get all the attention because they're model-controlled, but MCP's other
primitives carry half the design surface. The axis that organizes them: **who
controls the action** — model (tools), application/host (resources), user
(prompts, elicitation), or the host's own model (sampling, roots). Pick the
primitive by control flow, not by data shape.

## The map

| Primitive | Direction | Controlled by | Use it for |
| --- | --- | --- | --- |
| **Resources** | server → client | application | bulk/structured data the model should *read on demand*, not hold — documents, reports, file trees |
| **Resource templates** | server → client | application | parameterized URIs (`paniolo://stale/items/{id}`) for unbounded spaces |
| **Prompts** | server → client | user | canned workflows surfaced as slash commands — "fix all wiki findings" |
| **Sampling** | server → client (asks host's model) | host + user consent | server needs an LLM mid-operation without owning a model |
| **Elicitation** | server → client | user | structured input mid-request (forms in-context; URL mode for credentials/URLs) |
| **Roots** | client → server | client | filesystem/workspace boundaries the server should respect |
| **Tasks/subscriptions** | async messaging | both | long-running work and change streams (`subscriptions/listen`, `*/list_changed`) |

## Judgment calls

- **Resources vs tool results:** if the output is bigger than the model should
  carry or might be referenced again, it's a resource — return a
  `resource_link` from the tool instead of inlining. This is the cheapest
  token win on the whole surface ([mcp-token-efficiency](./mcp-token-efficiency.md)).
- **Prompts vs skills:** a prompt is a fixed, user-invoked recipe; a skill
  (SEP-2640) is model-loaded instructional context. Use prompts for
  deterministic "run this playbook" UX; skills when the model must decide
  when/how to apply the knowledge.
- **Sampling is the most uneven primitive** — several major hosts don't
  implement it, and where they do it's consent-gated. Design so a
  sampling-dependent feature degrades to "give the user the prompt" — never
  silently required.
- **Elicitation has two modes:** in-context (a form the user fills) and URL
  (the host opens a URL — for OAuth, credential entry, anything you must not
  see transit the model's context). Sensitive input goes through URL mode.
- **Roots are informational, not enforcement** — a client-supplied root is a
  hint about where the work lives; treat it like a search scope, not an
  authorization.

## The async substrate

The draft spec reframes long-running work around explicit handles: MCP is
**stateless** at the protocol layer — every request carries its `_meta`, no
connection implies a session — so anything multi-request needs an explicit
identifier (task IDs, subscription handles). `subscriptions/listen` returns a
long-lived response streaming notifications; `MRTR` covers multi-round-trip
requests where the server needs client input mid-call. If you're porting
operations that take minutes (batch fixes, agent runs), model them as tasks —
submit, poll, fetch — rather than holding a request open.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
