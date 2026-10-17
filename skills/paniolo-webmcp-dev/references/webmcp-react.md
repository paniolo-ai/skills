---
source-slug: webmcp-react
source-hash: 2acaf8b0bdbacedbbcf2ace18bfbdc6e0303f6843de863b82a9bdaa8b7f4f9ff
bundled: 2026-10-16
title: WebMCP in React
type: concept
tags:
- webmcp
- react
- hooks
updated: 2026-09-25
---

# WebMCP in React

Register tools with the component lifecycle: mount → `registerTool`, unmount or disable → unregister (`AbortSignal`). Chrome documents experimental React support via **`usewebmcp`**; Angular has a parallel experimental path.

## `usewebmcp` (tool-only)

Use when the app only needs `registerTool` ergonomics:

- Pair with native `document.modelContext`, `@mcp-b/webmcp-polyfill`, or `@mcp-b/global` (feature-detect; don't invent a fake global).
- Hook registers when enabled/mounted and cleans up on disable/unmount.
- Need prompts, resources, or MCP client providers → `@mcp-b/react-webmcp` instead.

## Patterns that matter

- **One tool set per editor surface** — register song-edit tools when the editor mounts; abort when the user navigates away ([webmcp-best-practices](./webmcp-best-practices.md)).
- **`execute` wraps existing app functions** — same path as UI handlers; pass `signal` into fetches.
- **State-dependent catalogs** — change which tools are registered when draft vs published mode changes; don't leave write tools up when the UI can't write.
- **Types** — `webmcp-types` for the imperative surface.

## Related

- [webmcp-apis](./webmcp-apis.md) — imperative API details
- [webmcp-proxy](./webmcp-proxy.md) — React `<WebMCPProxy />` for remote MCP bridging
- [webmcp-evals](./webmcp-evals.md) — include the full state-relevant tool list in isolation tests

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
