---
source-slug: web-component-lifecycle
source-hash: 33ae4814143d086233ae40ce6171530ab473a491765bd6b030e688c63518fad4
bundled: 2026-09-27
title: Web Component Lifecycle
type: concept
tags:
- web-components
- lifecycle
- custom-elements
updated: 2026-09-26
---

# Web Component Lifecycle

The browser calls lifecycle methods on your class — you never call them
yourself. Understanding what each callback guarantees (and forbids) is where
most custom-element bugs live.

## The callbacks

| Callback | Fires when | Use for |
| --- | --- | --- |
| `constructor()` | Element created or upgraded | `super()` first; attach shadow root; set private defaults |
| `connectedCallback()` | Added to the document | External listeners, data fetching, rendering setup |
| `disconnectedCallback()` | Removed from the document | Undo everything `connected` set up outside the element |
| `connectedMoveCallback()` | Moved via `Element.moveBefore()` | Preserve state on moves without disconnect→connect churn |
| `adoptedCallback()` | Moved to another document | Re-anchor document-scoped resources |
| `attributeChangedCallback(name, old, new)` | An `observedAttributes` entry changes | Side effects (ARIA sync); never set the matching property — see [web-component-api-design](./web-component-api-design.md) |

## Constructor rules (spec-mandated)

- Call `super()` first. Never `return` a value; don't use `document.write`,
  inspect attributes/children, or add attributes/children — children don't
  exist yet for parser-created elements.
- The constructor is the **only** place with exclusive knowledge of the
  element: attach the shadow root here, not in `connectedCallback` (the
  element may detach/reattach; constructor work happens once).

## connected/disconnected are paired, not once-only

Elements move (`insertBefore`, reparenting) — each move is a disconnect +
connect unless `moveBefore()` + `connectedMoveCallback` are used. Write both
callbacks as idempotent setup/teardown:

```js
connectedCallback() {
  super.connectedCallback?.();
  window.addEventListener('keydown', this._onKeydown);
}
disconnectedCallback() {
  window.removeEventListener('keydown', this._onKeydown);
}
```

Anything holding a reference to the element after disconnect (external
listeners, timers, observers) prevents GC. Listeners on the element's own
(shadow) DOM don't need removal — they die with it.

## Upgrade timing

`customElements.define` upgrades matching elements already in the DOM —
synchronously running `constructor` + `connectedCallback` on each. Two
consequences:

- **Properties set before upgrade** live on the instance and shadow your
  setters — capture them in `connectedCallback` (`_upgradeProperty`,
  [web-component-api-design](./web-component-api-design.md)). Lit's constructor does this for you.
- **Unupgraded elements are `display: inline` with no behavior** — style
  `:not(:defined)` or use Declarative Shadow DOM
  ([web-component-shadow-dom](./web-component-shadow-dom.md)) to avoid FOUC.

## Library note: Lit's update cycle

Lit layers an async, batched reactive-update cycle on top: property changes
schedule a microtask update; `render()` must be pure (input = properties,
no side effects); `connectedCallback` triggers the first update and
`disconnectedCallback` pauses it — but updates continue for a
previously-connected element regardless of connection state. When extending
Lit callbacks, always call `super` ([web-component-libraries](./web-component-libraries.md)).

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
