---
source-wiki: sharp-shooter-wiki
source-slug: typescript-update-guards-cover-every-gated-field
source-hash: 8a70d27e520d265e9a2eca8cc289b6e0b8683a96fb51c905accae3fc467554fd
bundled: 2026-08-15
title: Update Guards Cover Every Gated Field
type: concept
tags:
- authoring
- typescript
- state
updated: 2026-08-15
---

# Update Guards Cover Every Gated Field

A predicate that decides *whether to apply an incoming update* is a gate over the whole payload. If
it compares a subset of the fields it gates, every change confined to the uncompared fields is
silently dropped.

```ts
// ❌ Rejects any update where only the panel columns changed
function shouldApply(current: LiveRow | undefined, next: LiveRow): boolean {
	if (current === undefined) {
		return true;
	}
	if (next.slide_sequence > current.slide_sequence) {
		return true;
	}
	// Same sequence — assume nothing meaningful changed.
	return next.active_slide_position !== current.active_slide_position;
}

// ✅ Same-sequence deltas compare every field the guard gates
function shouldApply(current: LiveRow | undefined, next: LiveRow): boolean {
	if (current === undefined) {
		return true;
	}
	if (next.slide_sequence > current.slide_sequence) {
		return true;
	}
	// A sequence bump only tracks position. Panel and address columns change
	// without touching it, so they must be compared explicitly.
	return (
		next.active_slide_position !== current.active_slide_position ||
		next.panel_kind !== current.panel_kind ||
		next.panel_view_mode !== current.panel_view_mode ||
		next.active_slide_id !== current.active_slide_id ||
		next.active_slide_occurrence !== current.active_slide_occurrence
	);
}
```

## The rule

**Adding a synced field means editing the guard.** A column added to a table, a key added to a
payload, a property added to a state slice — each one has to be added to whatever predicate decides
that an update is worth applying. This is not automatic and the type system will not remind you,
because a comparison function that ignores a field is still well-typed.

Where a monotonic counter (`sequence`, `version`, `updated_at`) guards the update, be precise about
**what that counter actually tracks**. If it is bumped only by position changes, it cannot stand in
for "something changed" — it means "the position changed." Write that down in a comment next to the
guard; it is the assumption the next field addition will violate.

## Why it is expensive to find

The symptom is that updates never arrive. That points debugging at the transport — the subscription,
the connection, the server write — which is the wrong half of the system. The write succeeded, the
row arrived, the client looked at it and decided to ignore it.

In the case this page is drawn from, no broadcaster's mode change ever reached a follower, and the
project plan had already contained a card predicting precisely that failure.

## Test shape

Iterate the gated fields rather than picking one:

```ts
it.each([
	["panel_kind", { panel_kind: "calendar" }],
	["panel_view_mode", { panel_view_mode: "page" }],
	["active_slide_id", { active_slide_id: "slide-9" }],
	["active_slide_occurrence", { active_slide_occurrence: 2 }],
])("applies an update when only %s changed at an unchanged sequence", (_label, change) => {
	// Arrange
	const current = makeLiveRow();
	const next = { ...current, ...change };

	// Act
	const result = shouldApply(current, next);

	// Assert
	expect(result).toBe(true);
});
```

A table like this fails loudly when a field is added to the row and not to the guard, because the
new field has no row in the table and the omission is visible in review.

## See also

- [Exhaustive Switch Over Unions](typescript-exhaustive-switch-over-unions.md)
- [TypeScript (authoring) index](index.md)
