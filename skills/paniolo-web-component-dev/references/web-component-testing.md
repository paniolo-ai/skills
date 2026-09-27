---
source-slug: web-component-testing
source-hash: 276349f7711be596c7181f28688c77f037c29739ee09a089572f82fe22588086
bundled: 2026-09-26
title: Web Component Testing
type: concept
tags:
- web-components
- testing
- open-wc
updated: 2026-09-26
---

# Web Component Testing

Custom elements need a **real browser** (or faithful DOM) — shadow roots,
upgrades, slots, and `:defined` don't exist in a naive string-diffing setup.
The open-wc stack is the reference workflow; its patterns port to any
runner.

## Fixture + lifecycle timing

```js
import { fixture, expect } from '@open-wc/testing';
import '../src/a11y-input.js';

it('defaults label to empty string', async () => {
  const el = await fixture('<a11y-input></a11y-input>');
  expect(el.label).to.equal('');
});
```

`fixture()` stamps markup, registers, awaits the element's first update,
and returns the upgraded element. For Lit always `await el.updateComplete`
(or the fixture) before asserting rendered state — updates are async
microtasks.

## Assert semantics, not serialized DOM

Literal `shadowRoot.innerHTML` equality is a trap: Lit inserts `<!---->`
markers for dynamic parts, and whitespace/attribute ordering differs. Use
semantic DOM diffing (`@open-wc/semantic-dom-diff`):

```js
expect(el).shadowDom.to.equal(`
  <slot name="label"></slot>
  <slot name="input"></slot>
`);
expect(el).lightDom.to.equal(`<label slot="label">foo</label>`);
```

It normalizes both sides through the HTML parser and diffs the structure —
exactly what "is this rendered" means.

## What to test

- **Attributes ↔ properties:** set attribute → property reflects; set
  property → attribute reflects where promised
  ([web-component-api-design](./web-component-api-design.md)).
- **Events:** dispatch an internal action, assert `composed: true` event
  reaches an ancestor listener — the flag is easy to forget
  ([web-component-events](./web-component-events.md)).
- **Lifecycle:** connect → assert setup; disconnect → assert external
  listeners/observers released; reconnect → still works
  ([web-component-lifecycle](./web-component-lifecycle.md)).
- **Slots:** `slot.assignedElements()` and `slotchange` on content changes.
- **A11y:** axe-core style checks **with shadow DOM enabled**, plus a
  manual cross-root label/description check — automated tools historically
  under-test this ([web-component-accessibility](./web-component-accessibility.md)).
- **Forms:** `formData` contains your value; `setValidity` blocks
  submission; `form.reset()` hits `formResetCallback`
  ([web-component-forms](./web-component-forms.md)).

## Tooling

`@web/test-runner` (or Karma historically) runs tests in real browsers;
open-wc scaffolds the whole setup (`npm init @open-wc`). In a Playwright
world, prefer component tests that mount the element in a real page over
jsdom — jsdom has partial shadow DOM support and no upgrade semantics
worth trusting.
