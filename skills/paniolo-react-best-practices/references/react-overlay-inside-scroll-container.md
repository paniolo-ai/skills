---
source-wiki: sharp-shooter-wiki
source-slug: react-overlay-inside-scroll-container
source-hash: 2dd51323905fb8d2fbb260c80f76baf76b7e8cd73f3ddababbf5a11c9b95f100
bundled: 2026-08-15
title: Interactive Overlays Belong Inside the Scroll Container
type: concept
tags:
- authoring
- react
- client
- dom
updated: 2026-08-15
---

# Interactive Overlays Belong Inside the Scroll Container

An overlay that covers a scrollable element must be a **descendant of that element**, not a sibling
positioned on top of it. A sibling intercepts the pointer, and because it is not inside the
scroller, the browser finds no scrollable ancestor to act on.

```tsx
// ❌ Zones cover the scroller but are not inside it:
//    touch drag does not scroll, wheel goes elsewhere,
//    and the scroller's own handlers never fire
<div className="relative h-full">
	<div ref={containerRef} className="h-full overflow-y-auto">
		{children}
	</div>
	<button className="absolute inset-y-0 left-0 w-1/3" onClick={back} />
	<button className="absolute inset-y-0 right-0 w-1/3" onClick={forward} />
</div>

// ✅ Zones live inside the scroller, in a zero-height sticky layer
<div className="relative h-full @container">
	<div ref={containerRef} className="h-full overflow-y-auto" onTouchStart={…}>
		<div className="sticky top-0 z-40 h-0 overflow-visible">
			<button className="absolute top-0 left-0 h-[100cqh] w-1/3" onClick={back} />
			<button className="absolute top-0 right-0 h-[100cqh] w-1/3" onClick={forward} />
		</div>
		{children}
	</div>
</div>
```

The `h-0` sticky wrapper is the trick: it consumes no document height, so it does not disturb
layout or scroll extent, while its absolutely positioned children overflow into view. Size them
against the **non-scrolling** wrapper — a container query unit (`100cqh`) rather than `100%`, which
would resolve against the scroller's full content height.

## What breaks, and why it is hard to see

All of these fail at once, and none of them points at the overlay:

- **Touch drag does not scroll.** The touch starts on an element with no scrollable ancestor.
- **Wheel events go to the wrong element.**
- **Handlers on the scroll container never fire** — `onTouchStart`, `onPointerDown`, and anything
  built on them, such as a multi-touch gesture hook.

The reported symptom is usually "scrolling doesn't work in this mode," which sends debugging to
`overflow` rules and touch-action, not to the DOM nesting of an unrelated-looking button.

It is especially easy to miss when a **sibling implementation of the same UI works**: if another
mode renders the zones as children of a non-scrolling container, both versions look alike in review
and only one of them is correct.

## Checklist

- Is the overlay a descendant of the element that scrolls?
- Does it add document height it should not? Use `h-0` plus absolute children.
- Are the zones sized against the viewport-sized wrapper, not the scrolled content?
- If parts of the overlay should not take the pointer at all, `pointer-events-none` on those parts
  is the fix — not moving the whole layer outside.

## See also

- [Observer Snapshots and Late Mounts](react-observer-snapshots-and-late-mounts.md)
- [React (authoring) index](index.md)
