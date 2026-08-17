---
name: paniolo-react-compiler
description: |
  React Compiler (babel-plugin-react-compiler) constraints — no useCallback/useMemo/memo, no ref writes or other side effects during render, no try/finally or value blocks inside compiled hooks. Use when authoring or editing React components and hooks in a project that enables the React Compiler, or when diagnosing a ReactCompilerError or a dev-build panic. Do NOT use for projects without the compiler enabled — manual memoization is legitimate there; load paniolo-react-best-practices for compiler-independent React conventions.
license: MIT
metadata:
  version: 0.1.0
tags:
- react
- react-compiler
references:
- 'wiki: sharp-shooter-wiki:react-compiler → references/react-compiler.md'
- 'wiki: sharp-shooter-wiki:react-compiler-control-flow → references/react-compiler-control-flow.md'
- 'wiki: sharp-shooter-wiki:react-compiler-no-render-side-effects → references/react-compiler-no-render-side-effects.md'
---

**Requires:** file-read, terminal (lint / dev build). No network access needed.

**Applies only when the project enables the React Compiler.** Confirm before applying — check
`vite.config.ts` (or the equivalent bundler config) for `babel-plugin-react-compiler`, or
`package.json` for the dependency. If it is absent, this skill does not apply and its prohibitions
are wrong for that repo.

## Preconditions

- Confirm `babel-plugin-react-compiler` is enabled; note the configured `panicThreshold`.
- Read the component/hook file before editing.

## Defaults (proceed without asking)

- Apply all key rules below to compiled functions — default-export components and `use*` hook
  bodies.
- **Always ask:** before adding `memo` for a measured performance exception.

## Key rules

- **No manual memoization** — the compiler handles identity stability. Do **not** add `useCallback`
  or `useMemo`. Use plain `function handle(): void {}` handlers and plain `const` locals for derived
  values. Avoid `memo` unless there is a documented, measured exception.
  [react-compiler](references/react-compiler.md)

- **No side effects during render** — writing `ref.current` during render is a `ReactCompilerError`,
  not a warning. Assign it in `useLayoutEffect` instead, so the ref is current before any effect
  that reads it. The same rule bans in-place mutation of props/state values (`sort`, `reverse` —
  copy first), module-level `let` writes, and store setters called from render.
  [react-compiler-no-render-side-effects](references/react-compiler-no-render-side-effects.md)

- **Compiler-safe control flow** — do not use `try { … } finally { … }` in compiled hooks. Avoid
  ternaries, `&&` / `||` chains, and optional chaining in expression positions *inside* a `try`;
  hoist them to `const` assignments above it, or move the async work to a plain sibling `.ts` helper.
  [react-compiler-control-flow](references/react-compiler-control-flow.md)

- **Stability belongs to the consumer, not the producer.** When a callback needs a stable identity,
  do not memoize the producer — guard in the consumer (compare against a previous value held in a
  ref) or let the effect depend on the callback and accept the re-subscribe.
  [react-compiler-no-render-side-effects](references/react-compiler-no-render-side-effects.md)

- **`panicThreshold: "all_errors"` fails the build, not just lint.** Typecheck alone will not catch
  a compiler panic. Run lint or a dev build after editing hooks that contain `try`/`catch`, refs, or
  async saves.
  [react-compiler-control-flow](references/react-compiler-control-flow.md)

## Output format

Write code changes directly. After edits, list which compiler constraints applied and which
validation command was run.

## Error handling

- On a `ReactCompilerError`, report the message verbatim, then fix the cause — do not disable the
  plugin, lower `panicThreshold`, or add an `eslint-disable`.
- If a task appears to require `useCallback` or `useMemo`, flag it and ask before adding it; the
  answer is almost always a consumer-side guard instead.

## Validation

```bash
<your lint command>
```

## Evaluations (I/O examples)

**Input:** "This hook re-subscribes its IntersectionObserver on every render — stabilize the
callback with useCallback."
**Expected:** Declines the `useCallback`, explains the compiler forbids it, and instead holds the
callback in a ref assigned inside `useLayoutEffect` so the observer effect can keep an empty dep
array. Runs lint.

**Input:** "Build fails with `ReactCompilerError: Ref values may not be written during render`."
**Expected:** Locates the render-phase `ref.current = …` assignment, moves it into a
`useLayoutEffect` with the assigned value in the dep array, runs lint, reports the fix.

**Input:** "Wrap this expensive filter in useMemo."
**Expected:** Declines, replaces with a plain `const`, and notes the compiler memoizes it.

**Input:** "Add a useEffect that syncs `selectedId` to localStorage."
**Expected:** Defers to `paniolo-react-best-practices` for the useEffect comment and dependency
conventions; this skill only adds that no memoization is introduced.

## Skill handoffs

- Compiler-independent React conventions (props, file layout, `useEffect` comment rule, paired
  hooks) → load `paniolo-react-best-practices`.
- Hook tests → load `paniolo-vitest-test-best-practices`.
- TypeScript-only files with no React imports → load `paniolo-typescript-best-practices`.

## Do Not

- Do not apply this skill to a project without `babel-plugin-react-compiler` enabled.
- Do not add `useCallback` or `useMemo`.
- Do not write to a ref, mutate props/state, or call a setter during render.
- Do not use `try/finally` inside compiled hooks.
- Do not lower `panicThreshold` or disable the plugin to make an error go away.

## References

- [react-compiler](references/react-compiler.md) — memoization prohibition and rare exceptions
- [react-compiler-no-render-side-effects](references/react-compiler-no-render-side-effects.md) —
  render purity, ref writes, `useLayoutEffect`
- [react-compiler-control-flow](references/react-compiler-control-flow.md) — `try`/`catch` and
  value-block constraints
- [paniolo-react-best-practices/SKILL.md](../paniolo-react-best-practices/SKILL.md)
