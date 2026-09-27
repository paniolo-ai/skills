---
source-slug: web-component-events
source-hash: a37f60be62a51c6c1d97373fda7ddefe95c2c4c55a6faceb1dc40df907bb81a7
bundled: 2026-09-27
title: Web Component Events
type: concept
tags:
- web-components
- events
- dom
updated: 2026-09-26
---

# Web Component Events

Custom elements are `EventTarget`s — events are the standard upward channel
in the **"data down, events up"** pattern: parents set attributes/properties
down the tree; elements dispatch events that bubble up.

## When to dispatch — and when not to

- **Dispatch for internal activity** the host can't otherwise observe:
  timers finishing, resources loading, user interaction inside the shadow
  tree, internal state transitions.
- **Never dispatch in response to the host setting a property.** The host
  already knows — and data-binding systems will loop: host sets prop →
  element fires event → framework re-renders → sets prop again.
- Follow DOM event conventions: lowercase or kebab-case names
  (`change`, `item-selected`) — some frameworks' declarative bindings
  can't express uppercase event names ([web-component-framework-interop](./web-component-framework-interop.md)).
  Detail payload goes in `event.detail` via `CustomEvent`.

## Crossing the shadow boundary

Events composed inside a shadow root need explicit flags:

```js
this.dispatchEvent(new CustomEvent('item-selected', {
  detail: { id },
  bubbles: true,   // ascend ancestor chain
  composed: true,  // cross the shadow boundary into the document
}));
```

- Without `composed: true`, the event dies at the shadow root — the classic
  "my listener on the parent never fires" bug. Native UI events (click,
  input, focus in composed form) are composed; your `CustomEvent` is not by
  default.
- **Retargeting:** listeners outside see `event.target` as the *host*, not
  the internal element — that's encapsulation working, not a bug. Inspect
  `event.composedPath()` to see the real origin.
- **Focus:** `focus`/`blur` don't bubble; inside shadow DOM use `focusin`/
  `focusout` (composed) or `attachShadow({ delegatesFocus: true })` to
  forward host focus to the first focusable child.

## Checklist rules

- Events named like the platform (`change`, `input`, `close`) where a native
  analogue exists.
- Document the event surface like the rest of the API
  ([web-component-api-design](./web-component-api-design.md)): name, when it fires, `detail` shape.
- Prefer events over callback props (`onChange={fn}` attributes) — TAG:
  avoid callbacks; events compose through the tree for free.
