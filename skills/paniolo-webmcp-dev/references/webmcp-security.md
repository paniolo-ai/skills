---
source-slug: webmcp-security
source-hash: 8987126a8792059a274b69c11850443f30a19bd90990da3b3dc324d965e4080d
bundled: 2026-09-25
title: WebMCP Security
type: concept
tags:
- webmcp
- security
- agents
- prompt-injection
updated: 2026-09-25
---

# WebMCP Security

A WebMCP tool runs **in the page with the user's session and cookies**. That is
the product value and the risk: every registered tool is an entry point operated
by a model that reads untrusted content. Browser mitigations protect the user
from tools; protecting your system from a confused or injected agent stays on
the same server-side boundary you already had.

**Rule:** if a capability would be unsafe as a public endpoint called by a
hostile client, it is unsafe as a WebMCP tool. Auth, ownership, quotas, and rate
limits remain server-side.

## Site-side: annotation hints

Set these on `registerTool` annotations:

| Hint | Use when |
| --- | --- |
| `readOnlyHint: true` | Tool does not mutate app or system state |
| `untrustedContentHint: true` | Output includes UGC or external data |
| `consequentialHint: true` | High-stakes or non-reversible actions |
| `debugging: true` | DevTools / test tools (Chrome 156+); end-user agents may filter these |

## Site-side: origin exposure

By default other origins cannot observe or call your tools. Cross-origin use
requires **both**:

1. `exposedTo: ['https://partner.example']` on `registerTool` (secure origins)
2. `getTools({ fromOrigins: ['https://your.example'] })` on the caller

Cross-origin iframes also need Permissions Policy: `allow="tools"`. Only expose
tools to origins you would trust with that user data or action. Extensions with
`host_permission` can still script the page with or without WebMCP.

## Character budgets (Chrome guidance)

Keep descriptions and outputs short so agents do not hit guardrails: ~500 chars
per tool description, ~150 per parameter description, ~30 per name, ~1.5K per
tool output. Prefer fewer than ~40 tools on a page ([webmcp-apis](./webmcp-apis.md)).

## Agent-side defenses

Agents that consume WebMCP should assume tool definitions and outputs are
untrusted:

- **Deterministic** — token limits on tool responses; origin allowlists; user
  confirmation for mutating tools unless `readOnlyHint` is set; honor
  `untrustedContentHint` in system instructions
- **Probabilistic** — spotlighting (delimiters or Base64) so tool output is data,
  not instructions; critics that check intent alignment without seeing untrusted
  content; classifiers on manifests and outputs

Attack vectors called out by Chrome: malicious manifests (instructions hidden in
names/descriptions) and contaminated outputs (UGC carrying injection).

## Digests

- blog-vietanh-webmcp-attack-surface — lived attack-surface walkthrough
- blog-damiangalarza-webmcp-without-mcp-server — boundary vs MCP framing
