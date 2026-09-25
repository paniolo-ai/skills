---
source-slug: webmcp-user-journeys
source-hash: 57c974f8123bf0b1c89ab7c711629bfdc8d2db69a1f280cc4bd7a2fd17e49d1a
bundled: 2026-09-25
title: WebMCP User Journeys
type: concept
tags:
- webmcp
- product
- journeys
- cuj
updated: 2026-09-25
---

# WebMCP User Journeys

Design tools from **outcomes and conversations**, not from your API surface.
Chrome's CUJ guidance and the webmcp.com journey essay agree: wrap every
endpoint and you get a confused agent; work backwards from what the site exists
to produce and you get a coherent agent path.

## Two journey types

| Journey | Human on page? | Trust / tools |
| --- | --- | --- |
| **Co-browsing** | Yes — assistant rides along | Can span Answer → Action → Sensitive |
| **Remote execution** | Optional / none | Prefer Answer-only until governance matures |

WebMCP's stated design is co-browsing. Remote callers are technically possible;
treat them as opt-in (bot protection, throttling) and design deliberately.

## Three tool layers

| Layer | Risk | Examples |
| --- | --- | --- |
| **Answer** | Low | Search, availability, policies, draft state |
| **Action** | Medium | Add to cart, apply filter, save draft, hold slot |
| **Sensitive** | High | Book, buy, publish, cancel — confirmation / `consequentialHint` |

Ship Answer tools first. Map layers to journeys: co-browsing can use all three
with the human watching; remote should stay Answer-heavy.

## Build framework (Chrome)

1. **User goal** — ideal outcome, required context, boundaries, prioritize
   journeys where conversation beats UI.
2. **Initial state** — app location, agent context, auth/filters. Only register
   tools valid in that state ([webmcp-best-practices](./webmcp-best-practices.md)).
3. **Role-play** — turn-by-turn; at each turn list info, actions, tools, and UI
   reactions. Handle vague asks (agent requests missing params).
4. **Fail gracefully** — actionable errors ("search first", "date YYYY-MM-DD"),
   not silent fails or raw stack traces.
5. **Eval + telemetry** — journeys become [webmcp-evals](./webmcp-evals.md) fixtures; logs feed
   tool refinements ([webmcp-observability](./webmcp-observability.md)).

## Patterns from Chrome CUJs

- **Shopping** — `search_products` → wishlist/cart → refine; reorder via history.
- **Forms** — declarative `toolname` / populate / submit; split "start process"
  from field-filling tools.
- **Filter-heavy catalogs** — structured `search` + `apply_filters`; return
  agent-usable structure (ranked lists, map pins), not only HTML.

For an in-app editor: outcome = successful edit the user accepts; Answer =
read draft/sections; Action = apply content/structure changes; Sensitive =
publish/save-commit if irreversible. Prefer co-browsing tools wrapping the
editor UI ([webmcp-vs-mcp](./webmcp-vs-mcp.md)).
