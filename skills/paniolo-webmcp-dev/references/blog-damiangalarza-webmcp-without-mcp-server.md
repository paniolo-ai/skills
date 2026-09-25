---
source-slug: blog-damiangalarza-webmcp-without-mcp-server
source-hash: d9ac91d8c81ff8ce99d487eb77e9d01e7f388a6b6fd8f6b4ce6d21587b722e1f
bundled: 2026-09-25
title: Damian Galarza — WebMCP Without a Separate MCP Server
type: entity
tags:
- webmcp
- blog
- mcp
updated: 2026-09-25
---

# Damian Galarza — WebMCP Without a Separate MCP Server

[WebMCP: Agent-Ready Sites Without a Separate MCP
Server](https://www.damiangalarza.com/posts/2026-09-02-webmcp-agent-ready-without-separate-mcp-server/)
(Damian Galarza, 2026-09-02) is the clearest product framing of when in-page
tools replace a hosted MCP server — and when they do not.

## Takeaways

- WebMCP removes the **integration middleware**, not the **trust boundary**.
  Tools ride the existing session; server auth/ownership/quotas still say no.
- A tool is four parts: name, description, input schema, `execute` wrapping
  code the UI already calls. Feature-detect `document.modelContext`.
- Decision order: (1) is a capable browser tab open? (2) attended vs unattended?
  (3) does the work need live page state? (4) does the logic already exist
  client-side? (5) maybe both, sharing one service layer.
- Early ecosystem notes (late Aug 2026): Chrome OT, ChatGPT "Site tools"
  (imperative, top-level), Shopify Liquid storefront tools, API still moving.

Feeds [webmcp-overview](./webmcp-overview.md) (WebMCP vs MCP table) and [webmcp-security](./webmcp-security.md).
