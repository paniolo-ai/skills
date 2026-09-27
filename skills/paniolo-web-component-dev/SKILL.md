---
name: paniolo-web-component-dev
description: |
  Author, review, debug, or test web components — custom elements extending HTMLElement, usually with shadow DOM. Use when creating a custom element, choosing between attributes/properties/events/slots/CSS hooks, wiring form participation via formAssociated + ElementInternals, fixing cross-root ARIA or non-composed event bugs, styling with :host/::part/custom properties, choosing vanilla vs Lit vs Stencil, or making elements work inside React/Vue/Angular/Svelte apps. Do NOT use for components that only ever live inside one framework's render model with no cross-framework or no-JS requirement — that's framework component work, not custom element work.
license: MIT
metadata:
  version: 0.1.0
tags:
- web-components
- custom-elements
- shadow-dom
references:
- references/web-component-accessibility.md
- references/web-component-api-design.md
- references/web-component-best-practices.md
- references/web-component-events.md
- references/web-component-forms.md
- references/web-component-framework-interop.md
- references/web-component-libraries.md
- references/web-component-lifecycle.md
- references/web-component-overview.md
- references/web-component-shadow-dom.md
- references/web-component-styling.md
- references/web-component-testing.md
---

**Requires:** file-read, file-edit, terminal (local verification). A real
browser required for meaningful verification — jsdom cannot be trusted for
shadow DOM/upgrade semantics.

**Full reference:** [web-component-overview](references/web-component-overview.md) ·
[web-component-api-design](references/web-component-api-design.md) ·
[web-component-best-practices](references/web-component-best-practices.md)

Custom elements are platform primitives, not a framework. The quality bar is
"indistinguishable from a built-in element": instantiable from markup alone,
self-contained, keyboard-usable, themable. Every claim below traces to
web.dev's custom-element checklist, the W3C TAG design guidelines, or the
Gold Standard checklist.

## Preconditions

- Confirm this needs to be a custom element at all: cross-framework design
  system, framework-free embeddable, or standards-longevity argument → yes;
  app-internal component inside one framework → probably not.
  [web-component-overview](references/web-component-overview.md)
- Decide vanilla vs library **before** writing: leaf element, few
  attributes, static shadow tree → vanilla is fine; dynamic templating,
  reactive state, many properties → Lit (~5 KB, no build required).
  [web-component-libraries](references/web-component-libraries.md)

## Defaults (proceed without asking)

- Autonomous custom element (`extends HTMLElement`) — never customized
  built-ins (`is="…"`); Safari doesn't support them.
- `attachShadow({ mode: 'open' })` **in the constructor**; closed roots are
  deterrence not security and break tests/devtools.
- Tag name: hyphenated, library-prefixed (`acme-button`, not `button-x`).
- **Always ask:** which styling surface is intended to be public
  (`::part`/custom properties vs closed internals) if the repo's design
  system doesn't already establish it — parts are a forever-API.

## Key rules

### API surface (most design mistakes live here)

- **Attributes for primitives, properties for rich data.** Accept string/
  number/boolean as attribute *and* property; reflect between them unless
  high-frequency. Objects/arrays are properties only — never serialize to
  attributes.
- **Booleans are presence-based** (`hasAttribute`, not `disabled="false"`).
- **No reentrancy loops:** never set a property from
  `attributeChangedCallback` while the setter reflects to that attribute.
  Shape: setter writes attribute, getter reads it; the callback handles
  side effects (ARIA sync) only.
- **Capture pre-upgrade properties** in `connectedCallback`
  (`_upgradeProperty` delete-and-restore pattern) — frameworks bind before
  `define()` runs. Lit does this automatically.
- **Don't stamp author-set globals** — check `hasAttribute` before applying
  `role`/`tabindex`; `class` on the host is author-owned.
- **Data down, events up.** Dispatch for internal activity only — never in
  response to the host setting a property (binding loops). Kebab-case
  names, `bubbles: true, composed: true` to escape the shadow root.
  [web-component-api-design](references/web-component-api-design.md) ·
  [web-component-events](references/web-component-events.md)

### Lifecycle

- Constructor: `super()` first; attach shadow root; no attribute/child
  inspection, no DOM writes — children don't exist yet for parser-created
  elements.
- `connectedCallback`/`disconnectedCallback` are paired setup/teardown, not
  once-only — moves re-fire both unless `connectedMoveCallback` +
  `moveBefore()` are used. Everything external set up in `connected` must
  be released in `disconnected` or the element can't be GC'd.
- `attributeChangedCallback` only fires for `static observedAttributes`.
  [web-component-lifecycle](references/web-component-lifecycle.md)

### Styling

- `:host { display: block|inline-block|flex }` always — default `inline`
  makes width/height no-ops. Pair with `:host([hidden]) { display: none }`
  or your display rule beats the UA's `hidden`.
- Theming = CSS custom properties (shared/internal values) + `::part`
  (semantic internal elements). Parts can't nest and can't be removed
  without a breaking change. Default styling stays generic — no baked-in
  Material/Cupertino.
- Never style via author-facing attributes (`my-el[open]`) — reflection
  desync fails silently. Use internal classes or `:state()`.
  [web-component-styling](references/web-component-styling.md)

### Accessibility and forms (where naive ports die)

- **IDs are scoped per shadow root** — `aria-labelledby`/`for`/`aria-controls`
  across the boundary fails silently. Keep label+control in one root, use
  `ElementInternals` for default role/states, `ariaLabelledByElements`-style
  element refs where supported, `delegatesFocus` for focus forwarding.
  [web-component-accessibility](references/web-component-accessibility.md)
- **Forms need FACE:** `static formAssociated = true` +
  `attachInternals()` → `setFormValue`/`setValidity` + the four
  `form*Callback`s. A plain custom element inside `<form>` submits nothing.
  [web-component-forms](references/web-component-forms.md)

### Framework interop

- Modern React (≥19), Vue, Angular, Svelte, Lit all bind correctly **if**
  you follow the attr/prop/event rules above — both-way primitive
  reflection is what makes every framework row green. Kebab-case events for
  Vue's declarative bindings; rich data needs `.prop`-style property
  binding. React < 19 needs wrappers (`@lit/react`).
  [web-component-framework-interop](references/web-component-framework-interop.md)

## Output format

Write code changes directly. After edits, state which API surfaces were
added/changed (attributes, properties, events, slots, parts, custom
properties), what reflection/upgrade handling exists, and how you verified
the element in a real browser.

## Error handling

- Element renders nothing with no console error → check registration order
  (`define` before markup use), `observedAttributes` spelling, and whether
  an un-upgraded instance is hiding (style `:not(:defined)` while
  debugging).
- Parent listener never fires → missing `composed: true` on the event.
- `setAttribute` loop/stack overflow → property↔attribute reentrancy; move
  to setter-writes-attribute shape.
- Element invisible to forms/AT despite correct markup → not
  form-associated, or cross-root ID reference failing — see the two
  reference pages above.

## Validation

```bash
# No custom-element linter exists; verify in a real browser:
#   1. Serve the page; DevTools → Elements shows #shadow-root
#   2. el.matches(':defined') === true (upgrade happened)
#   3. el.shadowRoot.querySelector(...) returns internals
#   4. Dispatch a real interaction; ancestor listener fires (composed)
#   5. For FACE: new FormData(form).has(name); form.reset() resets it
# Tests: @open-wc/testing fixture + semantic-dom-diff (never raw innerHTML
# string equality — Lit inserts <!----> markers).
```

## Evaluations (I/O examples)

**Input:** "Build a `<my-toggle>` custom element with checked state"
**Expected:** Autonomous element, open shadow root in constructor, `checked`
as presence-based boolean attribute reflected both ways via
setter-writes-attribute/getter-reads-attribute, `change`-style
`bubbles+composed` event on user toggle (not on host property set), `:host`
display + `[hidden]` rule, `delegatesFocus` or internal keyboard handling,
`role`/`aria-checked` via ElementInternals (author-overridable).

**Input:** "My `aria-labelledby` inside the shadow root doesn't point at the
external label"
**Expected:** Explains ID scoping per shadow root; offers same-root
co-location, ElementInternals `ariaLabel`/element-ref APIs, or
host-forwarded attribute — not an IDREF hack.

**Input:** "Should we rewrite our React `<Button>` as a custom element?"
**Expected:** Asks about cross-framework/no-framework consumption first;
if app-internal only, advises keeping the React component rather than
adding an attr/prop/event translation layer.

## Skill handoffs

- React-side integration details (wrappers, `< 19` shims) →
  [paniolo-react-best-practices/SKILL.md](../paniolo-react-best-practices/SKILL.md).
- General TypeScript conventions →
  [paniolo-typescript-best-practices/SKILL.md](../paniolo-typescript-best-practices/SKILL.md).
- Vitest setup for non-browser unit tests →
  [paniolo-vitest-test-best-practices/SKILL.md](../paniolo-vitest-test-best-practices/SKILL.md);
  real-browser component testing per
  [web-component-testing](references/web-component-testing.md).

## Do Not

- Do not extend built-in elements (`HTMLButtonElement` + `is=`) — Safari
  will never support it; composition is the portable path.
- Do not attach the shadow root in `connectedCallback` or re-create it
  unconditionally — DSD-hydrated roots get wiped; check
  `internals.shadowRoot` first.
- Do not dispatch events in response to property sets — downward data flow
  needs no event and loops under data binding.
- Do not serialize objects/arrays into attributes or reflect rich data —
  properties only.
- Do not skip `disconnectedCallback` cleanup of external
  listeners/timers/observers.
- Do not expose `::part` casually or style on author attributes.
- Do not assume `aria-labelledby`/`for` work across shadow boundaries.
- Do not verify with jsdom alone — shadow DOM and upgrade semantics need a
  real browser.

## References

- [web-component-overview](references/web-component-overview.md) — the three specs, when to use
- [web-component-best-practices](references/web-component-best-practices.md) — canonical checklist
- [web-component-api-design](references/web-component-api-design.md) — attrs/props/events/slots/CSS hooks
- [web-component-lifecycle](references/web-component-lifecycle.md) — callbacks, upgrade timing
- [web-component-shadow-dom](references/web-component-shadow-dom.md) — encapsulation, slots, DSD
- [web-component-styling](references/web-component-styling.md) — `:host`, `::part`, custom props
- [web-component-events](references/web-component-events.md) — composed/bubbles, retargeting
- [web-component-accessibility](references/web-component-accessibility.md) — cross-root ARIA, internals
- [web-component-forms](references/web-component-forms.md) — FACE, setFormValue, validation
- [web-component-framework-interop](references/web-component-framework-interop.md) — per-framework binding
- [web-component-libraries](references/web-component-libraries.md) — vanilla vs Lit vs Stencil
- [web-component-testing](references/web-component-testing.md) — open-wc fixtures, semantic diffs

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
