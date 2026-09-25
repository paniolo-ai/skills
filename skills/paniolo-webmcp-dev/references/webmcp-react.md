---
source-slug: webmcp-react
source-hash: d4e1bb036f2d6af5ce95fd8e524c984ea210a5ce2e001d5c1faedbbfba33320c
bundled: 2026-09-25
title: WebMCP in React
type: concept
tags:
- webmcp
- react
- hooks
updated: 2026-09-25
---

# WebMCP in React

Register tools with the component lifecycle: mount → `registerTool`, unmount or
disable → unregister (`AbortSignal`). Chrome documents experimental React
support via **`usewebmcp`**; Angular has a parallel experimental path.

## `usewebmcp` (tool-only)

Use when the app only needs `registerTool` ergonomics:

- Pair with native `document.modelContext`, `@mcp-b/webmcp-polyfill`, or
  `@mcp-b/global` (feature-detect; don't invent a fake global).
- Hook registers when enabled/mounted and cleans up on disable/unmount.
- Need prompts, resources, or MCP client providers → `@mcp-b/react-webmcp`
  instead.

## Patterns that matter

- **One tool set per editor surface** — register song-edit tools when the editor
  mounts; abort when the user navigates away ([webmcp-best-practices](./webmcp-best-practices.md)).
- **`execute` wraps existing app functions** — same path as UI handlers; pass
  `signal` into fetches.
- **State-dependent catalogs** — change which tools are registered when draft
  vs published mode changes; don't leave write tools up when the UI can't write.
- **Types** — `webmcp-types` for the imperative surface.

## Related

- [webmcp-apis](./webmcp-apis.md) — imperative API details
- [webmcp-proxy](./webmcp-proxy.md) — React `<WebMCPProxy />` for remote MCP bridging
- [webmcp-evals](./webmcp-evals.md) — include the full state-relevant tool list in isolation tests
