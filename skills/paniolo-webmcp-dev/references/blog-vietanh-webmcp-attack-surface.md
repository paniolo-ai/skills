---
source-slug: blog-vietanh-webmcp-attack-surface
source-hash: 8e80cffa16595802c9396d15591c322d551215701fdf5653df181377cfe323d6
bundled: 2026-09-25
title: Viet-Anh — WebMCP Attack Surface
type: entity
tags:
- webmcp
- blog
- security
updated: 2026-09-25
---

# Viet-Anh — WebMCP Attack Surface

[I gave my website tools for AI agents with WebMCP, and thought hard about the
attack surface](https://www.vietanh.dev/blog/2026-07-06-webmcp-agent-ready-website)
(Viet-Anh, 2026-07-06; corrections through 2026-08) documents shipping WebMCP on
a personal site and probing what the browser actually mediates.

## Takeaways

- Status at write time: CG draft; Chrome OT / `--enable-features=WebMCPTesting`;
  Firefox/Safari uncommitted. Origin-trial tokens alone were not enough on stock
  Chrome without the testing feature flag.
- API renamed from `navigator.modelContext` to `document.modelContext` (CG
  moved the getter to `Document`).
- With the testing flag, `modelContext` is a real object — you can
  `registerTool` / `getTools` / `executeTool` from the console without an agent.
- Cross-origin visibility is dual-consent: `exposedTo` plus caller
  `fromOrigins`; iframes need `allow="tools"`.
- Treat every tool as a new attack surface: prompt injection via descriptions
  and outputs, over-privileged tools, schemas that invite sensitive inputs.
  Server-side auth and ownership still decide what the session may do.

Pairs with [webmcp-security](./webmcp-security.md) and Chrome's secure-tools / agent-security docs
in `raw/webmcp/`.
