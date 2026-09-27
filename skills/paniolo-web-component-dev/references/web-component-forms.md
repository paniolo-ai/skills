---
source-slug: web-component-forms
source-hash: 8a348d48f374a5283300a3841b54633998db27bfd5a1258a5493f0d02c64956d
bundled: 2026-09-27
title: Form-Associated Custom Elements
type: concept
tags:
- web-components
- forms
- elementinternals
updated: 2026-09-26
---

# Form-Associated Custom Elements (FACE)

A plain custom element inside a `<form>` is invisible to it: no submission,
no validation, no reset. **Form-associated custom elements** close that gap
via `static formAssociated = true` + `ElementInternals` — the element then
behaves like a built-in control: auto-associates with the nearest form,
works with `<label>`, submits a value, participates in validation, and gets
`:disabled`/`:valid`/`:invalid` styling.

## The minimal contract

```js
class MyCounter extends HTMLElement {
  static formAssociated = true;   // must be autonomous (extends HTMLElement)

  constructor() {
    super();
    this.internals = this.attachInternals(); // ElementInternals handle
  }

  // Mirror built-in control surface:
  get form()              { return this.internals.form; }
  get name()              { return this.getAttribute('name'); }
  get type()              { return this.localName; }
  get validity()          { return this.internals.validity; }
  get validationMessage() { return this.internals.validationMessage; }
  get willValidate()      { return this.internals.willValidate; }
  checkValidity()         { return this.internals.checkValidity(); }
  reportValidity()        { return this.internals.reportValidity(); }
}
```

`name` comes from the attribute and is the submitted key — don't override it.

## Values and validation

- `internals.setFormValue(value, state?)` — accepts a string, `File`, or
  `FormData` (multiple values under derived names: `n + '-first'`). Call it
  whenever the internal value changes. `null`/omitted ⇒ excluded from
  submission.
- `internals.setValidity(flags, message?, anchor?)` — set `{customError:
  true}` with a message; `{}` clears. Element then matches `:invalid`,
  blocks submission, and reports through the standard UI.
- `state` arg of `setFormValue` is for **restore-after-navigation**
  (autocomplete/af) — distinguish *value submitted* from *internal state*.

## Form lifecycle callbacks

`formAssociatedCallback(form)`, `formDisabledCallback(disabled)`,
`formResetCallback()`, `formStateRestoreCallback(state, mode)`. Implement
`formResetCallback` to restore defaults (browsers call it on
`form.reset()`) and `formDisabledCallback` to mirror `:disabled` styling —
consumers legitimately expect both to work.

## The cheaper escape hatch: `formdata` event

If you only need *submission* (not validation/labeling), listen for the
`formdata` event on the form and `formData.append(key, value)` — no
custom-element plumbing required. Good for progressive enhancement or
non-element logic.

## Gotchas

- `attachInternals()` on a non-custom element, or twice, throws — call once
  in the constructor after `super()`.
- FACE must extend `HTMLElement` directly — no customized built-ins (Safari
  doesn't support them anyway; [web-component-overview](./web-component-overview.md)).
- Labeling: `<label for>` can't reach into shadow, but a `<label>` wrapping
  the host, or `internals.ariaLabel`/`labels` API, covers it — details in
  [web-component-accessibility](./web-component-accessibility.md).

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
