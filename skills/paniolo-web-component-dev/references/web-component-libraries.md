---
source-slug: web-component-libraries
source-hash: 94d24e60826691b5e287f0b3588fcfa033e911d10bd395a6100efffa5af66a32
bundled: 2026-09-27
title: Web Component Libraries
type: concept
tags:
- web-components
- lit
- stencil
- libraries
updated: 2026-09-26
---

# Web Component Libraries

Vanilla custom elements are low-level: no templating, no reactivity, manual
attribute↔property sync, manual pre-upgrade capture. Libraries exist to
remove exactly that boilerplate — they're compile/target choices, not
replacements for the platform contract.

## Vanilla first — when it's enough

A leaf element with a few attributes and a static shadow tree (tooltip,
badge, toggle) is fine in ~50 lines of vanilla JS. Reach for a library when
you need: templating with dynamic parts, batched reactive updates,
declarative property definitions, or form/ARIA wiring sugar.

## Lit — the default recommendation

`LitElement` extends `HTMLElement` and adds a reactive-update cycle on top
of the standard lifecycle ([web-component-lifecycle](./web-component-lifecycle.md)):

- **Reactive properties:** `static properties = { foo: {type: String} }`
  auto-wires observedAttributes, attribute→property sync, and pre-upgrade
  capture. Set `reflect: true` for property→attribute.
- **Templates:** `html\`…\`` tagged literals; updates touch only changed
  expressions — no VDOM diffing. `render()` must be pure: properties in,
  template out, no side effects.
- **Async updates:** batched at microtask timing; await `updateComplete`
  before asserting on rendered DOM.
- **Styles:** `static styles = css\`…\`` lands as constructable stylesheets
  (shared per class).
- **Size ~5 KB min+gzip**; no build step required in dev.
- SSR via `@lit-labs/ssr` → emits Declarative Shadow DOM
  ([web-component-shadow-dom](./web-component-shadow-dom.md)).

Lit caveat: extending a lifecycle callback requires `super.*` calls, and
`connectedCallback`/`disconnectedCallback` still own external-resource
setup/teardown — Lit's pause/resume doesn't remove listeners for you.

## Others worth knowing

- **Stencil** — compiler-first (TSX/decorators), generates the elements +
  lazy loader + framework wrappers; good for distributing a design system
  as a library. Heavier toolchain; output is standard custom elements.
- **FAST** (Microsoft) — similar space to Lit; strong design-token story.
- **Shoelace/Web Awesome, Ionic, Spectrum WC** — reference implementations
  more than starting points; read their source for API conventions
  (Cianfrani calls Shoelace the de facto gold standard for practice).

## Choosing

- Distributing a multi-framework design system → Lit or Stencil (Stencil if
  you want generated React/Vue wrappers).
- Internal app components inside one framework → probably don't need custom
  elements at all ([web-component-overview](./web-component-overview.md)).
- Embedding a widget on third-party sites → vanilla or Lit, shadow DOM is
  the point.

Whatever the library, the public contract is identical — attributes,
properties, events, slots, CSS hooks ([web-component-api-design](./web-component-api-design.md)) — and
[web-component-framework-interop](./web-component-framework-interop.md) results apply unchanged.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
