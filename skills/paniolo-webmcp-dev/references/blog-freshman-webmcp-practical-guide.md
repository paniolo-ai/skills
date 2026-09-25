---
source-slug: blog-freshman-webmcp-practical-guide
source-hash: 8ae3cfc3834b1b96b0887b93b12770aaa147d67a850131c80dbcfcf92a0136df
bundled: 2026-09-25
title: Freshman — WebMCP Practical Guide
type: entity
tags:
- webmcp
- blog
- guide
updated: 2026-09-25
---

# Freshman — WebMCP Practical Guide

[WebMCP: A Practical Guide to Agent-Ready Websites](https://freshman.tech/webmcp/)
(Freshman, updated 2026-09-09) walks through enabling WebMCP in Chrome, building
search/cart/checkout tools on a demo storefront, and choosing imperative vs
declarative registration.

## Takeaways

- Agents can already drive sites via DOM or screenshots; WebMCP replaces that
  guesswork with named tools and schemas so actuation is faster and more
  reliable when the site cooperates.
- Start with a read-only search tool, then add mutating cart/checkout tools with
  honest descriptions and annotations (`readOnlyHint`, consequential paths).
- Imperative API for app logic; declarative form attributes when the form *is*
  the capability. Handle tool lifetime with `AbortSignal` and cancellation in
  `execute`.
- Cross-origin tools need mutual opt-in (`exposedTo` + `fromOrigins`) and
  `allow="tools"` on iframes.
- Compare WebMCP to MCP and to browser automation: tab-bound, session-local,
  human-visible; not a substitute for headless MCP servers.
- Adoption today means Chrome OT / flags and a small consumer set — ship thin
  wrappers over existing functions so downside stays low.

Canonical concept pages: [webmcp-overview](./webmcp-overview.md), [webmcp-apis](./webmcp-apis.md),
[webmcp-security](./webmcp-security.md).
