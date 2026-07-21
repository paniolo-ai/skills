---
source-wiki: sharp-shooter-wiki
source-slug: inline-comments
source-hash: 7e9c1958642b8eb1f78866c27210ce01ee47edea4e702b47e4a4c3f1d7f85598
bundled: 2026-07-20
title: Inline `//` comments
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
sources:
- raw/harness-eng/songshare-ai-system/skill-code-comment-best-practices.md
---

# Inline `//` comments

Use `//` above:

- `useEffect` blocks
- Complex conditionals or non-obvious logic
- Performance trade-offs
- TODOs/FIXMEs (see format below)
- Section markers inside longer functions

**Placement:** always on the line(s) immediately _above_ the target code. Never on the same line as
code.

```ts
// ❌
const result = compute(); // expensive — only runs once

// ✅
// expensive — only runs once
const result = compute();
```

**TODO/FIXME format.** Always include enough context to act on without additional research. Use
JSDoc (with `@see`) when the comment includes a link:

```ts
// ✅ Actionable with context
// TODO: Replace with server-driven pagination once API supports it — PROJ-456
// FIXME: Race condition when filters change rapidly — needs debounce
// TODO: Remove this polyfill when we drop IE11 support (Q3 2026)

// ❌ No context, not actionable
// TODO: fix this
// FIXME: doesn't work sometimes
```

When a TODO contains a link, use JSDoc:

```ts
/**
 * TODO: Remove this workaround once upstream fixes the null handling.
 *
 * @see https://github.com/example/library/issues/1234
 */
```

**Section comments** inside longer functions — use `//` to mark logical blocks:

```ts
const processOrder = (order: Order): ProcessedOrder => {
	// Validate order data
	validatePaymentMethod(order.payment);

	// Calculate totals
	const subtotal = calculateSubtotal(order.items);
	const tax = calculateTax(subtotal, order.billingAddress);

	// Apply discounts
	const discount = calculateDiscount(order.customer, subtotal);

	return { subtotal, tax, discount };
};
```

**`//` and JSDoc ordering.** A JSDoc block must be immediately above the symbol it documents — no
blank lines or `//` comments between them. If you need an explanatory note, place it _above_ the
JSDoc with a single blank line between the `//` and the JSDoc:

```ts
// ❌ — // comment between JSDoc and symbol
/**
 * Does something important.
 *
 * @returns void
 */
// Internal note: this touches global state (do not refactor)
function doSomething(): void {}

// ✅ — // comment above the JSDoc
// Internal note: this touches global state (do not refactor)

/**
 * Does something important.
 *
 * @returns void
 */
function doSomething(): void {}

// ✅✅ — preferred: integrate the note into JSDoc
/**
 * Does something important.
 *
 * @remarks
 * Touches global state — do not refactor without tracing all callers.
 *
 * @returns void
 */
function doSomething(): void {}
```

## See also

- Code comments (authoring) index