---
source-slug: webmcp-overview
source-hash: 39cdea8a02f26ac4cb97b398dcd1c4c569fda2ca956c30b16553820f49987ebe
bundled: 2026-09-25
title: WebMCP Overview
type: concept
tags:
- webmcp
- browser
- agents
- tools
updated: 2026-09-25
---

# WebMCP Overview

WebMCP is a proposed browser API that lets a web page expose client-side
actions as **tools** — named functions with natural-language descriptions and
JSON Schema inputs — to AI agents running in the browser (built-in agents,
extensions, or cross-origin iframes the page opts into).

The live surface is `document.modelContext`. Pages that use it behave like
MCP servers implemented in client script rather than on a backend: the agent
calls tools by name and schema; the browser mediates discovery and execution;
`execute` runs in the page with the user's existing session while the UI stays
visible.

It is a **Draft Community Group Report** under the W3C Web Machine Learning CG
(not a W3C Standard). Chrome runs an origin trial from Chrome 149 and a local
testing flag (`chrome://flags/#enable-webmcp-testing`). The API relocated from
`navigator.modelContext` to `document.modelContext`; feature-detect before use.

## What it is not

- **Not MCP over the wire** — tools only; no resources, prompts, JSON-RPC, or
  transport. Chrome calls the APIs "MCP-inspired."
- **Not discovery** — tools exist only while a capable browser has the page
  open. There is no crawlable manifest.
- **Not headless automation** — design intent is attended, in-tab workflows.
  Cron jobs, CLI agents, and remote services cannot call WebMCP tools.

## When WebMCP vs a dedicated MCP server

Decide **per capability**, not once for the product. Full comparison and
decision order: [webmcp-vs-mcp](./webmcp-vs-mcp.md). Short version: live tab + page/session state
→ WebMCP; unattended / CLI / anywhere-anytime → MCP; both can share one service
layer.

## Related pages

- [webmcp-vs-mcp](./webmcp-vs-mcp.md) — MCP vs WebMCP and how to decide what goes where
- [webmcp-vs-automation](./webmcp-vs-automation.md) — WebMCP vs DOM automation vs MCP Apps
- [webmcp-user-journeys](./webmcp-user-journeys.md) — CUJs and Answer / Action / Sensitive layers
- [webmcp-timeline](./webmcp-timeline.md) — Chromium OT/ship estimates, mobile, other engines
- [webmcp-apis](./webmcp-apis.md) — imperative `registerTool` / declarative form attributes
- [webmcp-implementation-gotchas](./webmcp-implementation-gotchas.md) — silent-failure bugs seen shipping a real tool
- [webmcp-react](./webmcp-react.md) — React `usewebmcp` lifecycle
- [webmcp-best-practices](./webmcp-best-practices.md) — tool strategy, naming, schemas, reliability
- [webmcp-evals](./webmcp-evals.md) — isolation and journey evaluations
- [webmcp-security](./webmcp-security.md) — hints, origin gating, agent defenses
- [webmcp-cross-origin](./webmcp-cross-origin.md) — cross-origin dual-consent gates
- [webmcp-observability](./webmcp-observability.md) — production metrics without PII dumps
- [blog-freshman-webmcp-practical-guide](./blog-freshman-webmcp-practical-guide.md) — end-to-end site guide
- [blog-vietanh-webmcp-attack-surface](./blog-vietanh-webmcp-attack-surface.md) — production attack-surface notes
