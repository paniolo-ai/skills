---
source-slug: webmcp-vs-mcp
source-hash: f2ca36b557460e8bcdb70d4077a5338e6e3efa95c9595d8709fba9ed104e64eb
bundled: 2026-09-25
title: MCP vs WebMCP
type: concept
tags:
- webmcp
- mcp
- architecture
- decision
updated: 2026-09-25
---

# MCP vs WebMCP

WebMCP does **not** replace MCP. Chrome's framing is partners, not opponents:
shared vocabulary (tools, schemas, intent), different homes for the capability.
Decide **per capability**, not once for the whole product.

## Side by side

| | MCP server | WebMCP |
| --- | --- | --- |
| Where it runs | Server you host | Client page (`document.modelContext`) |
| Lifecycle | Persistent | Ephemeral — gone when the tab closes |
| Who can call | Any MCP client (CLI, cloud, desktop, IDE) | Browser-integrated agents only |
| Auth | OAuth / API keys you operate | User's existing browser session |
| UI | Headless / external (MCP Apps host UI) | Same live site: DOM, cookies, visible UI |
| Protocol surface | Tools, resources, prompts, transports | Tools only ("MCP-inspired") |
| Discovery | Client registration / known servers | Visit the page; no crawlable manifest |

Also distinct from **browser automation** (DOM/screenshots when a site exposes
no tools) and **MCP Apps** (server-rendered UI inside an MCP host). WebMCP
keeps *your* website as the interface.

## Decision order

Ask these in order for each capability:

1. **Is a WebMCP-capable tab open while the task runs?**  
   No → MCP (or a plain API). Cron, Claude Code, cloud workers cannot call
   WebMCP tools.
2. **Attended or unattended?**  
   Unattended / batch → MCP. Attended, human-visible session → WebMCP is in
   scope (design intent is oversight; browsers lean on confirmation for
   sensitive actions).
3. **Does the work need live page or session state?**  
   Cart, unsaved draft, filtered dashboard, forms the user is looking at →
   WebMCP. Pure system-of-record reads/writes with no tab context → MCP.
4. **Where does the logic already live?**  
   Thin wrap of existing client functions → cheap WebMCP. Logic only behind a
   server API you would expose anyway → MCP may be nearly as cheap and reaches
   more clients.
5. **Need both?**  
   Share one service layer: MCP for headless clients; WebMCP tools on the page
   that call the same operations. Optional: [webmcp-proxy](./webmcp-proxy.md) can register a
   remote MCP server's tools onto `document.modelContext` for browsing agents.

## What goes where (examples)

| Capability | Prefer | Why |
| --- | --- | --- |
| Checkout with live cart | WebMCP | Session + visible UI |
| Fill warranty / support form on-site | WebMCP | DOM-aware, user watches |
| Nightly reporting / sync jobs | MCP | No tab; unattended |
| IDE or CLI agent managing records | MCP | Non-browser clients |
| Catalog search from any agent host | MCP | Anywhere, anytime |
| Catalog search while shopping in-tab | WebMCP (or both) | Same logic; page context |

## Security boundary does not move

Whether the entry point is MCP or WebMCP, **auth, ownership, quotas, and rate
limits stay server-side**. A WebMCP tool rides cookies; that is not a new trust
model — if it would be unsafe as a hostile public endpoint, it is unsafe as a
tool. Annotations and origin gating: [webmcp-security](./webmcp-security.md).

## Related

- [webmcp-overview](./webmcp-overview.md) — what WebMCP is
- [webmcp-proxy](./webmcp-proxy.md) — bridge an existing MCP server onto `document.modelContext`
- [webmcp-best-practices](./webmcp-best-practices.md) — how to shape tools once you choose WebMCP
- [blog-damiangalarza-webmcp-without-mcp-server](./blog-damiangalarza-webmcp-without-mcp-server.md) — longer product framing
- [blog-freshman-webmcp-practical-guide](./blog-freshman-webmcp-practical-guide.md) — WebMCP vs MCP vs automation table
