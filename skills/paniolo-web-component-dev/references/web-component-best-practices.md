---
source-slug: web-component-best-practices
source-hash: 1ed05f40b186a036ece9ece068038ffaab4ea80a1d89f62d62a35cf6591c0f68
bundled: 2026-09-27
title: Web Component Best Practices
type: concept
tags:
- web-components
- best-practices
- checklist
updated: 2026-09-26
---

# Web Component Best Practices

The bar to aim for is **indistinguishable from a built-in element**: a custom
element should be instantiable from markup alone, self-contained,
keyboard-usable, and unsurprising to theme. Details live on the topic pages;
this page is the checklist that ties them together.

## Behave like HTML

- **Never throw from markup use.** Native elements don't validate their
  content model; a `<div>` inside an `<img>` fails silently. Match that —
  degrade gracefully, don't pollute the console (TAG + Cianfrani).
- **Self-contained.** Ship all dependencies; resolve resource paths relative
  to the component source; load in any order.
- **Detached instantiation works.** No `document`/`parentNode` assumptions
  before `connectedCallback`.
- **Detachable and reattachable.** `disconnectedCallback` releases external
  listeners; `connectedCallback` re-establishes. See [web-component-lifecycle](./web-component-lifecycle.md).

## Public API

- **Attributes for primitives, properties for rich data.** Every attribute
  ideally has a paired property; reflect between them for primitive values
  but never serialize objects to attributes. Full rules:
  [web-component-api-design](./web-component-api-design.md).
- **Boolean attributes are presence-checked** (`hasAttribute`), not
  `disabled="false"`.
- **Don't override author-set global attributes** — check `hasAttribute`
  before applying default `role`, `tabindex`, etc.
- **Data down, events up.** Dispatch events for internal activity; never in
  response to the host setting a property (loops with data binding). See
  [web-component-events](./web-component-events.md).
- **Name things like the platform does** — reuse existing attribute/state
  vocabulary (`selected`, `open`, `disabled`) instead of inventing synonyms.

## Encapsulation and styling

- **Attach a shadow root in the constructor** and put element-created
  children inside it. Project user children with `<slot>`. See
  [web-component-shadow-dom](./web-component-shadow-dom.md).
- **Set a `:host` display** (`block`, `inline-block`, `flex`) — the default
  `inline` makes `width`/`height` no-ops and surprises consumers.
- **Respect `hidden`**: `:host([hidden]) { display: none }`, because your
  `:host { display: … }` otherwise wins over the UA's `[hidden]` rule.
- **Don't self-apply classes on the host** — `class` belongs to the page
  author; express state with attributes (or `:state()`).
- **Basic default styling only**; expose theming via CSS custom properties
  and `::part` rather than attributes like `bgcolor`. See
  [web-component-styling](./web-component-styling.md).

## Composition and robustness

- **Prefer child elements over data blobs** for rich content —
  `<my-list><my-list-item>…` beats a serialized `data=` attribute
  (blog-cianfrani-web-component-best-practices).
- **Don't enforce parent/child relationships** unless real; elements should
  work under any parent.
- **Handle pre-upgrade properties.** Frameworks may set properties before
  `define()` runs; capture them in `connectedCallback` (the
  `_upgradeProperty` pattern — [web-component-api-design](./web-component-api-design.md)).
- **Interactive ⇒ focusable + keyboard-complete + labeled.** The Gold
  Standard's a11y section is non-optional — [web-component-accessibility](./web-component-accessibility.md).
- **Test in a real browser** with shadow-aware assertions —
  [web-component-testing](./web-component-testing.md).
