---
source-wiki: sharp-shooter-wiki
source-slug: react-compiler-control-flow
source-hash: 4a031506391592b1e5ff81078965d6f6f3c977aff2d101e1ceece1c2a00d8ed6
bundled: 2026-08-01
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

After editing hooks with async saves or `try/catch`, **verify with `npm run lint` or a dev build** —
typecheck alone will not catch compiler panics.

---

See also:

- TypeScript (authoring) index
- [Effect-TS — useEffect boundaries](effect-ts#react-useeffect-boundaries)

## See also

- React (authoring) index