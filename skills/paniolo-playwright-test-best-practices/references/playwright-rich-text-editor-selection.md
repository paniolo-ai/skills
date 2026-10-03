---
source-slug: playwright-rich-text-editor-selection
source-hash: 735f2a64dc5a3517290e14da2a5f17f26a9b67e95f309a0fb269f20e83cc23b1
bundled: 2026-10-02
title: Rich-Text Editor Selection in E2E Tests
type: concept
tags:
- authoring
- playwright
- testing
- e2e
- contenteditable
updated: 2026-09-30
---

# Rich-Text Editor Selection in E2E Tests

Selecting a stored-text range inside a ProseMirror or other contenteditable surface is a
two-layer problem. The editor's own state selection and the browser DOM `Selection` can diverge:
editor keymaps, commands, and confirmation gates read editor state, while typing, Backspace, and
paste `beforeinput` paths act on the DOM selection at input time. A test that verifies one layer
and drives the other flakes per engine — the editor can hold range `[0,1)` while the DOM caret
sits at `1`, so Backspace deletes the character after the target.

## Approaches that fail

- `Home` + `Shift+ArrowRight`: `Home` is a visual-line boundary, not a stored-text offset. When
  the editable region mixes text with `contenteditable="false"` widget decorations, each engine
  maps visual lines to stored offsets differently.
- Synthetic `Selection.setBaseAndExtent`, dispatched `selectionchange` events, or measured mouse
  drags: the editor only adopts a DOM selection while its surface is focused and its tracked
  selection permits it. Calling `focus()` can also restore a stale tracked range and suppress
  adoption for a window of frames.
- `getSelection().toString()` as the sole verification: WebKit can hold the correct editor-state
  range while the DOM selection string stays empty — a node selection over an inline-decorated
  span.

## A strategy that holds across Chromium, Firefox, and WebKit

1. Click the editable region, then verify `document.activeElement` is inside the surface — an
   intercepted or missed click leaves every later signal stale.
1. Normalize: press `ArrowRight` once to collapse any lingering range at its end through the
   editor's own handler.
1. Converge to the target range **end** with sequential real `ArrowLeft`/`ArrowRight` presses —
   each keypress must settle before the next.
1. Extend **backward** with `Shift+ArrowLeft`. Backward extension behaves consistently across
   engines; forward `Shift+ArrowRight` resolves differently at inline decoration boundaries.
1. If an engine collapses instead of extending (WebKit does this over wrapped inline
   decorations), call `getSelection().modify("extend", "backward", "character")` — it mutates
   the live DOM selection and fires a native `selectionchange` the editor adopts.
1. Verify both layers before any destructive key: the editor-state selection via an
   app-published signal, and the DOM range via a reader that walks only editable text nodes —
   skip `contenteditable="false"` subtrees so widget text does not corrupt offsets.
1. If the DOM range diverges from the intended stored range, re-park the caret at the range end
   and re-extend, with a bounded retry count. Poll rather than read once — the editor rewrites
   the DOM selection a frame after adopting state.

## Give the test an observable selection signal

A collapsed-caret indicator cannot distinguish caret@0 from range `[0,1)`. Publish selection
state in a live region (`aria-live` + `data-testid`) that renders ranges distinctly — for
example `position N` for a caret and `position N–M` for a range. The helper can then assert
editor-state adoption directly instead of inferring it from DOM geometry.

## Related pitfalls

- Fixed or sticky headers can intercept `locator.click()` even after scrolling — Playwright's
  own `scrollIntoViewIfNeeded` may re-park the target under the header. `dispatchEvent("click")`
  bypasses hit-testing and still reaches delegated listeners.
- SPA read-after-write: navigating to a just-saved entity can render a stale prefetched copy.
  Assert the save request payload, then use a bounded reload-retry on the fresh view rather than
  a long timeout.
- Guarded-edit tests must produce an actually-consequential edit: if the product has a fast path
  that preserves anchors (for example rebinding a mark onto a same-width replacement), a minimal
  payload takes the fast path and no confirmation ever parks. Choose a payload the fast path
  cannot handle.
- `page.evaluate` and `locator.evaluate` callbacks are serialized into the page — module-scope
  constants are not in scope inside the callback. Declare them inside the callback or pass them
  as arguments, and use `evaluateHandle` when you need live DOM node references.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
