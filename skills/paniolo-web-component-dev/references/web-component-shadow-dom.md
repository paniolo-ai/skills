---
source-slug: web-component-shadow-dom
source-hash: 8eaa1980fcf3435eaf9cb426080f43dbadcc3813c54a87ed16c6fb18dbb71c91
bundled: 2026-09-27
title: Web Component Shadow DOM
type: concept
tags:
- web-components
- shadow-dom
- slots
- ssr
updated: 2026-09-26
---

# Shadow DOM

Shadow DOM attaches an encapsulated tree to an element: internals are hidden
from page JS and CSS, IDs are scoped per-root, and `<slot>` projects the
author's light-DOM children into chosen positions. It's what lets a custom
element be dropped into any page without breaking or being broken.

## Attach it in the constructor

The constructor is the only moment with exclusive knowledge of the element.
Attaching later (`connectedCallback`) forces guards for detach/reattach.
Place implementation children in the shadow root; leave author-supplied
children in light DOM so they remain visible/accessible even if the element
never upgrades.

```js
constructor() {
  super();
  this.attachShadow({ mode: 'open' }).innerHTML = `<slot></slot>`;
}
```

## Open vs closed

- **`mode: 'open'`** (default choice): `el.shadowRoot` is reachable — needed
  for testing, legitimate tooling, and your own debugging.
- **`mode: 'closed'`**: `el.shadowRoot` is `null`. This is deterrence, not
  security — internals are still reachable via refs kept at attach time, and
  it breaks tests/devtools. Rarely worth it; the web.dev checklist assumes
  open.

## Slots and light DOM

- `<slot>` projects light-DOM children into the shadow tree; named slots
  (`<slot name="label">` + `<span slot="label">`) route children to specific
  positions; default slot catches the rest.
- Slotted content keeps its light-DOM parent — it *inherits* `lang`/`dir`
  from the light parent but *renders* with the flattened tree's direction.
- `slotchange` events on `<slot>` (or `slot.assignedElements()`) are how you
  observe runtime changes to distributed content — the Gold Standard
  requires responding to them.
- Elements that create their own children (list items, icons) are
  implementation details → shadow root, never light DOM.

## Declarative Shadow DOM (SSR)

`<template shadowrootmode="open">` as a host's first child produces a real
shadow root at parse time — no JS required; Baseline since Aug 2024
(`shadowrootmode` is the standardized spelling; `shadowroot` was the older
Chrome-only one).

Hydration: an upgraded element may already have a root. Don't blind-call
`attachShadow` — check first via `ElementInternals.shadowRoot` (works for
closed roots too), then fall back:

```js
const internals = this.attachInternals();
const shadow = internals.shadowRoot ?? this.attachShadow({ mode: 'open' });
```

Calling `attachShadow` on an element with a *declarative* root doesn't
throw — it empties and returns that root, which keeps older components
working but would wipe SSR'd content; hence the check.

## What encapsulation costs you

- IDs are scoped per root → cross-root `aria-labelledby`-style references
  fail. That's the accessibility boundary: [web-component-accessibility](./web-component-accessibility.md).
- Events must opt in to crossing the boundary (`composed: true`):
  [web-component-events](./web-component-events.md).
- Page CSS can't reach in except via custom properties / `::part`:
  [web-component-styling](./web-component-styling.md).

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
