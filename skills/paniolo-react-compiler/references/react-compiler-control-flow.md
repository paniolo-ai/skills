---
source-wiki: sharp-shooter-wiki
source-slug: react-compiler-control-flow
source-hash: b50bb9ba2c449705c5c1938b1db20dd086526605d2695290df858cbb4bb55e3c
bundled: 2026-08-16
title: Compiler-compatible control flow
type: concept
tags:
- authoring
- react
- client
updated: 2026-06-18
---

# Compiler-compatible control flow

Vite runs **`babel-plugin-react-compiler`** with **`panicThreshold: "all_errors"`** (see
`vite.config.ts`). Patterns the compiler cannot lower **fail the dev build** from the React Babel
plugin, not only ESLint or TypeScript.

This applies to **compiled functions** the pipeline infers — typically default-export **components**
and **`use*` hook** bodies.

- **Do not use `try { … } finally { … }` in compiled hooks/components.** The compiler does not
  lower `finally` yet. Prefer **`setIsSaving(false)` (or equivalent) on each exit path** before
  `return`, or move the whole async operation to a **plain sibling `*.ts` module** (not a hook) and
  `await` it from the hook so the hook stays thin.
- **Inside `try` / `catch`**, avoid complex **expression** forms the compiler rejects as “value
  blocks” inside the try: ternaries, `&&` / `||` chains, optional chaining, and similar nested logic
  **in expression positions**. **Hoist** normalization to **`const` assignments above `try`**, then
  keep the `try` block to mostly linear `await` and simple assignments.
- For non-trivial branching around network calls, prefer a **small extracted `async` function** in
  a separate file (returns a result discriminant or throws) so the hook does not embed a large
  `try/catch` surface.

After editing hooks with async saves or `try/catch`, **verify with lint or a dev build** —
typecheck alone will not catch compiler panics.

---

See also:

- TypeScript (authoring) index
- [Effect-TS — useEffect boundaries](effect-ts#react-useeffect-boundaries)

## See also

- React (authoring) index
