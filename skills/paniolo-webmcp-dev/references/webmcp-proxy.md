---
source-slug: webmcp-proxy
source-hash: 99164aa20a503216d10a0201ea4300a780a45d10eded778787cc1e793679e257
bundled: 2026-09-25
title: WebMCP Proxy
type: concept
tags:
- webmcp
- mcp
- proxy
- bridge
updated: 2026-09-25
---

# WebMCP Proxy

[`webmcp-proxy`](https://github.com/alpic-ai/webmcp-proxy) (Alpic) bridges an
**existing remote MCP server** onto the browser's WebMCP surface. Browsing agents
discover the server's tools via `document.modelContext` without you reimplementing
them as page-local `execute` callbacks.

This is one concrete way to do "both" from [webmcp-vs-mcp](./webmcp-vs-mcp.md): keep MCP as the
service layer, expose the same tools to in-tab agents.

## What it does

1. Connects to a remote MCP endpoint (Streamable HTTP, fallback SSE).
2. Lists tools (`tools/list`).
3. Registers each with `document.modelContext.registerTool`.
4. On agent invoke, proxies `tools/call` and returns the result.

```ts
import { createWebMcpProxy } from "webmcp-proxy";

const proxy = await createWebMcpProxy({
  url: "https://mcp.example.com/mcp",
  // headers: { Authorization: "Bearer …" },
});
// later: await proxy.disconnect();
```

React: `<WebMCPProxy url="…" />`. Vue package also available.

## When to use it

| Fit | Not a fit |
| --- | --- |
| You already have an MCP server and want browsing agents to call it | Capability only makes sense as live page/DOM state (cart UI, form focus) |
| Same tool catalog for CLI/cloud MCP clients and in-tab agents | No CORS / no Streamable HTTP or SSE on the server |
| Thin page glue; logic stays on the server | You need resources/prompts — WebMCP is still tools-only |

Page-native tools (session UI, declarative forms) still belong as local
`registerTool` / form attributes — see [webmcp-apis](./webmcp-apis.md) and
[webmcp-best-practices](./webmcp-best-practices.md). Proxy and local tools **coexist**: the library uses
`registerTool`, not `provideContext`, so it does not wipe page-owned tools.

## Requirements and caveats

- Browser must expose WebMCP (`document.modelContext`; deprecated
  `navigator.modelContext` fallback). Otherwise the proxy logs and no-ops.
- MCP server must allow **CORS** from the page origin.
- Chrome 153+: per-execution `AbortSignal` is forwarded to remote `tools/call`.
- Auth via optional `headers` — same trust rules as any browser-callable MCP
  endpoint; server still owns authorization ([webmcp-security](./webmcp-security.md)).
- Demo: [webmcp.netlify.app](https://webmcp.netlify.app/).

## Related

- [webmcp-vs-mcp](./webmcp-vs-mcp.md) — when MCP vs WebMCP vs both
- [webmcp-overview](./webmcp-overview.md) — what WebMCP is
- [webmcp-apis](./webmcp-apis.md) — native registration without a proxy
