---
source-wiki: sharp-shooter-wiki
source-slug: typescript-exhaustive-switch-over-unions
source-hash: c2e8b1e59f14a8abb8c77f4002a967b83acd1100ce92af10de6ba0818bcf2bfe
bundled: 2026-08-15
title: Exhaustive Switch Over Unions
type: concept
tags:
- authoring
- typescript
updated: 2026-08-15
---

# Exhaustive Switch Over Unions

When you `switch` on a discriminated union, give it a `default` that assigns the value to `never`.
Adding a variant then becomes a **compile error at every switch that does not handle it** — which is
the only reliable way to find them all.

```ts
type BroadcastAction =
	| { kind: "adopt-mode"; mode: PanelViewMode }
	| { kind: "slide-change"; position: number }
	| { kind: "song-change"; songId: string };

/**
 * Fail the build when a union variant is added without a handler.
 *
 * @param value - The value that should have been narrowed to `never`
 * @returns never
 */
export default function assertNever(value: never): never {
	throw new Error(`Unhandled variant: ${JSON.stringify(value)}`);
}

function applyAction(action: BroadcastAction): void {
	switch (action.kind) {
		case "adopt-mode":
			setMode(action.mode);
			return;
		case "slide-change":
			setPosition(action.position);
			return;
		case "song-change":
			setSong(action.songId);
			return;
		default:
			assertNever(action);
	}
}
```

## Why this matters more than it looks

The failure it prevents is **silent and looks like success**. A `switch` with no `default` that
handles one variant and ignores three compiles, runs, passes tests that only exercise the handled
path, and produces no error at runtime — the other cases are computed correctly and then dropped on
the floor.

That is exactly what happened in the case this page is drawn from: a broadcast-action taxonomy was
derived correctly by diffing live rows, and the dispatch loop executed one of its variants. Every
other action was discarded. Nothing in the type system, the linter, or the test suite objected.

The cost of the guard is one line and one shared helper. The cost of omitting it is proportional to
how long it takes someone to notice that a feature quietly does nothing.

## Where it applies

- Any `switch` over a discriminated union or a string-literal union.
- Reducers and dispatch loops especially — they are where taxonomies get enumerated, and where a
  dropped branch is least visible.
- Exhaustive mapping objects are an alternative: `Record<Action["kind"], Handler>` also fails to
  compile when a variant is added, and is often cleaner when every branch is a function call.

## Do not

- Do not write `default: break` or `default: return` to silence a lint rule about missing defaults —
  that is the bug this page exists to prevent, made permanent.
- Do not cast the value in the default branch to make the error go away. If `assertNever` does not
  compile, a variant is genuinely unhandled.

## See also

- TypeScript (authoring) index
