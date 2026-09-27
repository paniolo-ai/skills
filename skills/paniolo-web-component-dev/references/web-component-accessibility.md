---
source-slug: web-component-accessibility
source-hash: fbc162f56e8ba2f7dfc271cc8e11dea03f420a604c073a42eda9bf22fbf18ea1
bundled: 2026-09-26
title: Web Component Accessibility
type: concept
tags:
- web-components
- accessibility
- aria
- shadow-dom
updated: 2026-09-26
---

# Web Component Accessibility

Shadow DOM scopes IDs per root — so **any feature that references elements
by ID fails across the boundary**: `<label for>`, `aria-labelledby`,
`aria-describedby`, `aria-controls`, `aria-activedescendant`, `aria-owns`,
`aria-flowto`, `aria-details`, `aria-errormessage`. This is the single
hardest problem in web-component accessibility; nothing in the platform
bridges it automatically.

## What works today

- **`ElementInternals` ARIA defaults** — set default role/states on the host
  without clobbering author intent:

  ```js
  this.#internals = this.attachInternals();
  this.#internals.role = 'checkbox';
  this.#internals.ariaChecked = 'true';
  ```

  Author-set `role`/`aria-*` attributes override internals — the reverse of
  the [web-component-api-design](./web-component-api-design.md) "don't override the author" dance done
  right. Form-associated elements get this plus labeling for free
  ([web-component-forms](./web-component-forms.md)).

- **`ariaLabelledByElements`-style element references** — set *element*
  references instead of ID strings (`input.ariaLabelledByElements = [label]`),
  bypassing ID scoping. Caveat: the target must be in the same or an
  **ancestor** shadow root — the restriction prevents leaking closed-root
  internals. `ElementInternals` targets may cross any boundary in the same
  document. Safari shipped; Chromium/FF rolled out later — check support.

- **Keep relationships inside one root** where possible — a label and its
  control in the same shadow tree work fine. Sibling-component
  relationships (`<custom-label>` ↔ `<custom-input>`) are what break.

- **`delegatesFocus: true`** on `attachShadow` forwards host focus/Tab order
  to the first focusable internal element.

## What doesn't (yet)

- Reference Target for Cross-Root ARIA (declare a shadow element as the
  target for host-level ARIA) is the promising proposal — Chromium origin
  trial as of mid-2025; don't build on it in production.
- Janky text-copying (`aria-labelledby` → `aria-label` + MutationObserver)
  can't express `aria-controls`/`aria-activedescendant` and relies on
  accessible-name computation libraries that often don't support shadow
  DOM. Last resort.
- ARIA mixins don't work with Declarative Shadow DOM — element refs need JS
  to wire up, so SSR'd trees are inaccessible until hydration.

## Non-negotiables (Gold Standard)

- Interactive ⇒ keyboard-complete: Tab/Shift+Tab reachable, expected keys
  for the role (`Enter`/`Space`/arrows), visible focus indicator.
- Meaningful structure: DOM order matches the meaningful order; wrap or
  mirror native element semantics rather than rebuilding from `<div>`s.
- Labels: significant non-text elements get `aria-label`/`aria-labelledby`
  or `alt`; let the page author override defaults.
- Not color-only or sound-only; respect contrast and forced-colors mode.
- Fail silently like native elements — a misplaced child doesn't throw
  ([web-component-best-practices](./web-component-best-practices.md)).

## Practical pattern

For a labeled control: put the `<label>` and `<input>` in the **same**
shadow root, use `internals.ariaLabel` or an `aria-label` attribute the
author sets on the host (forward it), and let `<label for>`/`for` work only
where you control both ends. Test with a real screen reader — automated
checks don't cover AT traversal across shadow boundaries.
