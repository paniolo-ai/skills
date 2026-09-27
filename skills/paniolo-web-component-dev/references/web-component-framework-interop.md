---
source-slug: web-component-framework-interop
source-hash: 9c3f211ade9c965688c5eec53d8f8ab0f999413082e3a9d917bb4b1419c07e3b
bundled: 2026-09-27
title: Web Component Framework Interop
type: concept
tags:
- web-components
- frameworks
- react
- vue
- angular
updated: 2026-09-26
---

# Web Component Framework Interop

Frameworks consume custom elements fine *today* — the
custom-elements-everywhere suite runs each through basic+advanced tests —
but the binding contract differs per framework, and that's what your
element design must satisfy. Two things matter: **how data lands**
(attribute vs property) and **how events are heard**.

## Per-framework binding behavior

| Framework | Data to element | Events from element |
| --- | --- | --- |
| React ≥19 | Heuristic: property if defined on instance, else attribute | `on*` props attach listeners; all casings work |
| Vue 3 | Attributes by default; `:foo.prop="x"` forces property | Declarative: lowercase/kebab only; else imperative |
| Angular | Properties by default; explicit attr binding available | All casings |
| Svelte | Property if defined on instance, else attribute | All casings |
| Lit | Attributes by default; `.prop=${x}` forces property | All casings |
| Solid / Preact / Mithril | Property-defined heuristic | All casings |

## Design implications for element authors

- **Reflect primitives both ways.** Frameworks in the "attributes by
  default" camp (Vue, Lit) still work because users read state via
  `getAttribute`; "property-first" camps (Angular, Stencil) work because
  your properties exist. Both-way reflection is what makes all rows green —
  [web-component-api-design](./web-component-api-design.md).
- **Rich data requires property assignment.** Attribute-only consumers
  can't pass objects; document the `.prop`/`:foo.prop` escape hatch.
- **Use lowercase/kebab event names** — Vue and Polymer's declarative
  bindings can't express `camelCase`/`PascalCase` events. `item-selected`
  works everywhere; `itemSelected` doesn't in Vue templates.
- **Pre-upgrade properties are real.** Frameworks stamp + bind before your
  `define()` loads — implement the `_upgradeProperty` capture in
  `connectedCallback` ([web-component-api-design](./web-component-api-design.md)), or use Lit which does
  it automatically.
- **React < 19 was the bad one** (attributes only, synthetic event system
  missed `composed` events). For React < 19 support, ship wrappers
  (`@lit/react`'s `createComponent`) — v19+ needs none.

## Rule of thumb

Your element is the contract; frameworks are clients. If you follow the
attribute/property/event conventions in [web-component-best-practices](./web-component-best-practices.md),
every modern framework binds correctly without shims — and testing one
framework row of custom-elements-everywhere is a cheap regression check.
