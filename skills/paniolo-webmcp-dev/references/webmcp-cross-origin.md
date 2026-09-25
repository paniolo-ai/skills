---
source-slug: webmcp-cross-origin
source-hash: dfb63b0555eac9794fd5eb59443b90580e9417cf2e0ca7e6626e9e841ed72b05
bundled: 2026-09-25
title: WebMCP Cross-Origin Tools
type: concept
tags:
- webmcp
- security
- iframe
- origins
updated: 2026-09-25
---

# WebMCP Cross-Origin Tools

By default, tools are **same-origin only**. Cross-origin use is dual-consent:
the tool owner opts in, and the caller must ask for that origin by name.

Use it only when the tool's implementation or session **must** stay on another
origin **and** a browsing agent on this page must call it. Otherwise keep tools
same-origin.

## When cross-origin is appropriate

| Pattern | Why origins differ |
| --- | --- |
| **Embedded partner widgets** | Host page embeds payments, search, or catalog from a partner origin; agent needs the iframe's tools without the host owning that code |
| **Hosted agent chrome in an iframe** | Chat/agent UI on a vendor origin embedded on your site discovers tools on your document (page-agent style) |
| **Multi-brand / white-label shells** | Parent shell and product app live on different origins; shell agent needs a few tools from the embed |
| **Marketplace embeds** | Seller capabilities stay on seller origins inside a buyer/host page; host agent only sees tools sellers opted to expose |
| **Auth or session split** | Capability must run under origin B's cookies (SSO, PCI payment frame) while agent UX lives on origin A |

Same-site subdomains you can unify under one origin for the agent surface usually
do **not** need cross-origin WebMCP — prefer page-local registration there.

## Gates

1. **Origin isolation** — WebMCP requires origin-isolated documents. Disabling
   isolation (e.g. `Origin-Agent-Cluster: ?0` / `document.domain`) disables the
   API.
2. **Permissions Policy `tools`** — defaults to `self`. Cross-origin iframes
   need `allow="tools"` on the iframe (and matching policy headers if used).
3. **`exposedTo`** on `registerTool` — list of secure origins allowed to see
   and execute the tool.
4. **`fromOrigins` on `getTools`** — caller must list those origins
   explicitly. Secure origins only.

Missing any gate → tool invisible or unavailable. Extensions with
`host_permission` are a separate path (can script the page with or without
WebMCP) — see [webmcp-security](./webmcp-security.md).

## Practical rule

Only `exposedTo` origins you would trust with that user data or action. A
read-only "favorites" tool still leaks preference data; a write tool acts as
the user. Prefer same-origin page-local tools unless you have a real embed or
partner product.

## Related

- [webmcp-apis](./webmcp-apis.md) — `registerTool` / `getTools` options
- [webmcp-security](./webmcp-security.md) — hints and agent-side defenses
- [webmcp-user-journeys](./webmcp-user-journeys.md) — when co-browsing needs partner capabilities
