---
source-slug: web-component-styling
source-hash: d3c11771483437f6d0baa81f5afc306b490042af092db3b5f09c6240246204c5
bundled: 2026-09-27
title: Web Component Styling
type: concept
tags:
- web-components
- css
- shadow-dom
- theming
updated: 2026-09-26
---

# Web Component Styling

Styling a custom element is an **API design decision**, not an
implementation detail. Page CSS cannot cross the shadow boundary — so every
appearance knob you intend to offer must be deliberately exposed.

## The two-way encapsulation boundary

- Inside → outside: shadow styles never leak into the page.
- Outside → inside: page rules can't select shadow children. The only doors
  are **CSS custom properties** (inherited through the boundary) and
  **`::part`** (explicitly exported elements).
- Light-DOM children stay styleable by the page — they're the author's
  content, rendered into your slots. You can arrange them with `::slotted()`
  but only their top level: `::slotted(span)` works, `::slotted(span b)`
  doesn't.

## Host styling rules

- **Always declare `:host { display: … }`** (`block`/`inline-block`/`flex`).
  Custom elements default to `inline`; width/height silently no-op.
- **Re-declare `hidden`:** `:host([hidden]) { display: none; }` — your
  `:host` display rule otherwise beats the UA's `[hidden]` rule on cascade.
- **Don't style on author attributes.** If `open` isn't reflected perfectly,
  `my-el[open]` selectors break hard to debug. Style internal classes/state,
  or use `:state()` (`CustomStateSet` via ElementInternals) for
  state-selectors that can't desync.
- **Don't self-apply `class` on the host** — it belongs to the page author.
- **`:host-context(ancestor)`** exists for ancestor-based variants; use
  sparingly (it leaks page structure into your styling).

## Theming surface

- **CSS custom properties for shared values** — tokens reused internally
  (`--my-el-border-width`) or brand axes (`--my-el-accent`). Define defaults
  on `:host`: `color: var(--my-el-color, inherit)`.
- **`::part(name)` for structural styling** — expose semantically named
  internal elements (`part="button"`, `part="track"`). Constraints: parts
  can't nest, can't select children (`::part(x) > svg` fails), and every
  exposed part becomes a forever-API — a breaking change to remove. If an
  atom's whole body is a part, that's fine; don't blanket-part everything.
- **Sometimes the answer is "you don't."** Constrained variants
  (`type="primary"`) can be a better API than a styling hook.
- TAG baseline: ship **generic, subdued default styling** — functional, not
  themed; no Material/Cupertino baked in. Inherit font and colors by
  default.

## Common gotchas

- **FOUC/inline-before-upgrade:** `my-el:not(:defined) { opacity: 0 }` or
  Declarative Shadow DOM ([web-component-shadow-dom](./web-component-shadow-dom.md)) for SSR.
- **Constructable stylesheets** (`adoptedStyleSheets`) share one sheet
  across instances — cheaper than a `<style>` per instance for design
  systems.
- **Global page resets don't apply inside shadow** — set your own
  `box-sizing`, margins, fonts inside the root.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
