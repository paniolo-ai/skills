---
source-slug: web-component-api-design
source-hash: d4a2ad15557a393e9b43617b6ddc926d54f2c27e93ae5747a0b8ec15a4bc04a8
bundled: 2026-09-27
title: Web Component API Design
type: concept
tags:
- web-components
- api
- attributes
- properties
updated: 2026-09-26
---

# Web Component API Design

A custom element has **five** API surfaces, and picking the wrong one is the
most common design error: attributes, properties, methods, slots, and CSS
hooks (custom properties + `::part`). Your element's *styling* is part of
its API — treat it as such.

## Which surface for which job

| Surface | Reach | Cost / caveat |
| --- | --- | --- |
| Attributes | Declarative, works without JS | Strings only; primitives |
| Properties | Any JS value | Requires script; invisible in markup |
| Methods | Actions (`show()`/`hide()`); can return promises | JS only — prefer a matching attribute for state |
| Slots | Arbitrary HTML children | Author writes more markup; implementation complexity |
| CSS custom props | Theming tokens | Drowns DX if exposed per-property — use for shared values (`--border-width`) |
| `::part` | Style named internal elements | Public forever once shipped; can't nest or select children |

Rule of thumb from Cianfrani: choose per feature — a two-variant button
wants an attribute (`type="primary"`); arbitrary user markup wants a slot;
"how do I style XYZ" may correctly be *you don't*, or a `::part`, not a prop.

## Attributes vs properties (the core contract)

- **Primitives (string/number/boolean): accept as attribute *and* property.**
  Consumers set either; both should work.
- **Reflect primitives both ways** where not burdensome —
  `setAttribute('foo', v)` ↔ `el.foo`. Skip reflection for high-frequency
  values (`currentTime`) and **never reflect rich data to attributes**
  (serialization cost + lost references).
- **Rich data (objects, arrays): properties only.** No built-in element
  takes objects via attributes; don't invent the pattern.
- **Booleans: presence semantics.** `hasAttribute('open')`, not
  `open="false"`. Pair with a boolean property that adds/removes the
  attribute.
- **Declarative parity:** the TAG requires HTML and JS APIs stay linked —
  `<x-foo open>` and `el.open = true` must mean the same thing. Pick a
  source of truth (usually the attribute) and stick to it.

## Avoid reentrancy loops

Don't set a property from `attributeChangedCallback` while the property
setter reflects to the same attribute — infinite loop. The safe shape: the
setter writes the attribute; the getter reads it:

```js
set checked(value) {
  value ? this.setAttribute('checked', '') : this.removeAttribute('checked');
}
get checked() { return this.hasAttribute('checked'); }
```

`attributeChangedCallback` then handles *side effects* only (e.g.
`aria-checked`), never setting the property back.

## Handle pre-upgrade properties

Frameworks stamp markup and bind before `customElements.define` runs;
instance properties set pre-upgrade shadow the class's setters. Adopt them
in `connectedCallback`:

```js
_upgradeProperty(prop) {
  if (this.hasOwnProperty(prop)) {
    const value = this[prop];
    delete this[prop];
    this[prop] = value; // re-run through the real setter
  }
}
```

Lit does this automatically in its constructor; vanilla elements must do it
by hand. See [web-component-libraries](./web-component-libraries.md).

## Don't override the author

Check before stamping global attributes:

```js
connectedCallback() {
  if (!this.hasAttribute('role')) this.setAttribute('role', 'checkbox');
  if (!this.hasAttribute('tabindex')) this.setAttribute('tabindex', '0');
}
```

`tabindex="-1"` from the author is a deliberate signal — honor it. Same
principle for `class` (author-owned) and `id` on the host.

## Naming

Use existing platform vocabulary (`disabled`, `open`, `selected`, `value`)
before inventing terms; a `web-component-` prefixed skill or library should
read like it shipped with HTML. The hyphen in the tag name is mandatory —
it's the namespace reservation mechanism, so pick a library prefix
(`shoelace-` → `sl-`) deliberately.
