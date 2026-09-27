---
source-slug: web-component-overview
source-hash: a555e45f9e33aad2d9c99242d4cc2d17f8dd47620893b8a358bf528928e406e0
bundled: 2026-09-26
title: Web Components Overview
type: concept
tags:
- web-components
- dom
- standards
updated: 2026-09-26
---

# Web Components Overview

Web Components are a suite of **platform standards** — not a framework — for
defining reusable HTML elements with encapsulated behavior and styling. A
custom element is a real DOM element: it can be created from markup, script,
or any framework, and it interoperates with everything else that speaks DOM.

## The three specifications

- **Custom elements** — `customElements.define('my-el', class extends
  HTMLElement …)` registers a tag name (must contain a hyphen) with a class
  the browser instantiates. Two flavors: **autonomous** (extend
  `HTMLElement`; the normal choice) and **customized built-in** (extend e.g.
  `HTMLButtonElement` via `is="…"` — **Safari does not support these; avoid**
  for portable code).
- **Shadow DOM** — `attachShadow({mode})` attaches an encapsulated tree for
  markup/style/ID scoping; `<slot>` projects light-DOM children into it. See
  [web-component-shadow-dom](./web-component-shadow-dom.md).
- **HTML templates** — `<template>` holds inert markup for cloning or
  declarative shadow roots.

Modern additions worth knowing: **Declarative Shadow DOM** (`<template
shadowrootmode="open">`, Baseline since 2024 — SSR without JS),
**ElementInternals** (forms + default ARIA semantics — see
[web-component-forms](./web-component-forms.md) and [web-component-accessibility](./web-component-accessibility.md)), `connectedMoveCallback`
for state-preserving moves, and scoped `CustomElementRegistry` for
subtree-local tag definitions.

## When web components earn their keep

- **Design systems and shared widgets** consumed by multiple frameworks or
  no framework — this is the canonical use case (Shoelace, Ionic, Spectrum).
- **Leaf/embeddable UI** that must survive outside your stack's render cycle.
- **Long-lived** code: standards outlive framework versions.

## When to skip them

- App-internal components inside one framework — the framework's own model
  (props, slots, context) is cheaper and typed. Custom elements add an
  attr/prop/event translation layer for little gain.
- Rich-data-heavy UIs — attributes are strings; objects/arrays need
  properties, which frameworks must opt into ([web-component-framework-interop](./web-component-framework-interop.md)).
- Anything needing custom elements' missing semantics is real work — a11y
  relationships ([web-component-accessibility](./web-component-accessibility.md)) and forms
  ([web-component-forms](./web-component-forms.md)) are where naive ports break.

## Related pages

- [web-component-best-practices](./web-component-best-practices.md) — distilled canonical checklist
- [web-component-api-design](./web-component-api-design.md) — attributes vs properties vs events vs slots
- [web-component-lifecycle](./web-component-lifecycle.md) — callbacks, upgrade timing, constructor rules
- [web-component-shadow-dom](./web-component-shadow-dom.md) — encapsulation, slots, declarative SD
- [web-component-styling](./web-component-styling.md) — `:host`, `::part`, CSS custom properties
- [web-component-events](./web-component-events.md) — bubbling/composed flags, retargeting
- [web-component-accessibility](./web-component-accessibility.md) — cross-root ARIA, ElementInternals
- [web-component-forms](./web-component-forms.md) — form-associated custom elements
- [web-component-framework-interop](./web-component-framework-interop.md) — how each framework binds to CEs
- [web-component-libraries](./web-component-libraries.md) — vanilla vs Lit vs Stencil
- [web-component-testing](./web-component-testing.md) — fixture/shadow-DOM-aware assertions
- blog-cianfrani-web-component-best-practices — practitioner field notes
