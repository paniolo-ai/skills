---
source-wiki: sharp-shooter-wiki
source-slug: react-windowed-list-estimated-size
source-hash: 27332d6d1333738330b152a687b4fa13c6f89713c03bcc10e0e9e9e69f237e78
bundled: 2026-08-15
title: Windowed Lists Need an Estimated Size
type: concept
tags:
- authoring
- react
- client
- dom
updated: 2026-08-15
---

# Windowed Lists Need an Estimated Size

A windowed list replaces unmounted items with spacers that preserve scroll extent. The spacer's
height usually comes from a measurement cached when the item was last mounted — which means an item
that has **never** been mounted has no measurement, and its spacer renders at `0px`.

That is a deadlock. The scroll extent standing in for the item is what lets the reader scroll far
enough to mount it. At `0px` there is nothing to scroll through, so it never mounts, so it is never
measured.

```ts
const DEFAULT_ROW_HEIGHT_PX = 320;

/**
 * Height to reserve for an item that is not currently mounted.
 *
 * A measured height is authoritative. Without one, estimate — never fall back
 * to zero, which removes the scroll extent needed to reach the item at all.
 *
 * @param itemId - Item whose placeholder height is needed
 * @param measured - Heights captured while items were mounted
 * @param rowCount - Rows the item will render, used for the estimate
 * @returns Height in pixels to reserve
 */
function getSpacerHeight(
	itemId: string,
	measured: ReadonlyMap<string, number>,
	rowCount: number,
): number {
	return measured.get(itemId) ?? rowCount * DEFAULT_ROW_HEIGHT_PX;
}
```

## The rule

**A placeholder's fallback is an estimate, never zero.** Prefer a cheap estimate derived from the
data you already have — row count, character count, item type — over a single global constant, but
any positive number beats `0`.

A rough estimate is self-correcting: it is wrong only until the item mounts once, after which the
measurement takes over. Zero is not self-correcting, because it prevents the mount that would fix
it.

## Symptoms

- The list works for the first screenful or two and then will not scroll further.
- The scrollbar is much shorter than the item count implies.
- Jumping directly to a late item works, but scrolling to it does not — the strongest signal, since
  it shows the item renders fine and only the path to it is missing.

## See also

- [Observer Snapshots and Late Mounts](react-observer-snapshots-and-late-mounts.md)
- [React (authoring) index](index.md)
