---
source-slug: webmcp-apis
source-hash: f6c8b745ae50cc1e8ee92d46bf4c2199e5692e1b35fcfbd460ea69c422c54d3f
bundled: 2026-09-25
title: WebMCP Imperative and Declarative APIs
type: concept
tags:
- webmcp
- browser
- api
- react
updated: 2026-09-25
---

# WebMCP Imperative and Declarative APIs

Two registration shapes share one browser-mediated tool registry. Prefer the
imperative API for app logic; use declarative annotations when an ordinary HTML
form *is* the tool. Overview and product fit: [webmcp-overview](./webmcp-overview.md). Security
hints and origin gating: [webmcp-security](./webmcp-security.md).

## Imperative API

```js
await document.modelContext.registerTool({
  name: "search_products",
  description: "Search the catalog by free-text query.",
  inputSchema: {
    type: "object",
    properties: { query: { type: "string" } },
    required: ["query"],
  },
  annotations: { readOnlyHint: true },
  execute: async ({ query }, { signal }) => {
    const results = await searchCatalog(query, { signal });
    return JSON.stringify(results);
  },
});
```

| Method | Role |
| --- | --- |
| `registerTool(tool, options?)` | Register; optional `signal` to unregister, `exposedTo` for cross-origin |
| `getTools({ fromOrigins }?)` | List tools the caller may see (same-origin by default) |
| `executeTool(tool, input, options?)` | Run a discovered tool; optional cancel `signal` |

- Pass `AbortSignal` into long work inside `execute` so user/agent cancel stops
  fetches.
- As of Chrome 153, unregistering a tool does not abort in-flight executions.
- Listen for `toolchange` on `document.modelContext` when the set changes.
- JSON-stringified `executeTool` inputs are deprecated from Chrome 155 — pass a
  serializable object.

## Declarative API

Annotate a form; the browser derives name, description, and input schema:

- `toolname` / `tooldescription` on `<form>` (required; removing either
  unregisters)
- `toolparamdescription` on fields (optional; else label / `aria-description`)
- `toolautosubmit` to submit/navigate when the agent invokes the tool

`SubmitEvent` adds `agentInvoked` and `respondWith(Promise)` (call
`preventDefault()` first). Window events: `toolactivated`, `toolcancel`. CSS:
`:tool-form-active`, `:tool-submit-active`.

Some consumers (e.g. ChatGPT Site tools at early ship) supported only the
imperative API — treat declarative as complementary, not universal.

## React and ecosystem helpers

- **`usewebmcp`** — tool-only React hook over `document.modelContext` (pair with
  native support or `@mcp-b/webmcp-polyfill`). Use `@mcp-b/react-webmcp` when you
  need prompts/resources/client providers.
- **`webmcp-types`** — TypeScript typings for the imperative surface.
- **[webmcp-proxy](./webmcp-proxy.md)** — register tools from a remote MCP server onto the page
  without replacing page-local `registerTool` registrations.

## Tool count

Lighthouse warns above **40** registered tools. Register for current page
state; unregister with `AbortSignal` when the UI can no longer perform the
action.

## Before you ship

`document.modelContext` can attach to the page asynchronously — a one-shot
feature-detect at script load can miss it and silently register nothing.
Deployment can drop the registration script entirely with no error (an SPA
fallback masks it as a 200). See [webmcp-implementation-gotchas](./webmcp-implementation-gotchas.md) before
trusting a "no errors, tools just don't appear" result.
