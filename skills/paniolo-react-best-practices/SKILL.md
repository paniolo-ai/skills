---
name: paniolo-react-best-practices
description: |
  React 18+ conventions for this project — React Compiler (no manual memoization), ReactElement ambient type, useEffect comment rule, plain function declarations, file and import conventions. Use when authoring or editing any React component, hook, or page. Do NOT use for general TypeScript-only utilities with no React imports — load paniolo-typescript-best-practices instead.
license: MIT
metadata:
  version: 0.1.0
tags:
  - react
references:
  - "wiki: sharp-shooter-wiki:react → references/react.md"
  - "wiki: sharp-shooter-wiki:react-prefer-required-props → references/react-prefer-required-props.md"
  - "wiki: sharp-shooter-wiki:react-avoid-react-fc → references/react-avoid-react-fc.md"
  - "wiki: sharp-shooter-wiki:react-optional-render-nothing → references/react-optional-render-nothing.md"
  - "wiki: sharp-shooter-wiki:react-common-prop-types → references/react-common-prop-types.md"
  - "wiki: sharp-shooter-wiki:react-function-declaration-style → references/react-function-declaration-style.md"
  - "wiki: sharp-shooter-wiki:react-useeffect-rules → references/react-useeffect-rules.md"
  - "wiki: sharp-shooter-wiki:react-compiler → references/react-compiler.md"
  - "wiki: sharp-shooter-wiki:react-compiler-control-flow → references/react-compiler-control-flow.md"
  - "wiki: sharp-shooter-wiki:react-file-and-import-conventions → references/react-file-and-import-conventions.md"
  - "wiki: sharp-shooter-wiki:react-paired-hook-for-non-display-logic → references/react-paired-hook-for-non-display-logic.md"
---

**Requires:** file-read, terminal (linting/testing). No network access needed.

**Full reference:** [react](references/react.md)

**Companion skill (load on demand):**

- Zustand store selectors/slices →
  [zustand-best-practices/SKILL.md](../paniolo-zustand-best-practices/SKILL.md)

## Preconditions

- Read the component/hook file before editing.
- Check rules for repo-wide constraints.

## Defaults (proceed without asking)

- Apply all key rules below; edit the file already open or mentioned.
- **Always ask:** which file if not specified and cannot be inferred.

## Key rules

- **Prefer required props** — avoid many optional props; use noops/fallback refs at call sites
  instead of optional handlers.
  [react-prefer-required-props](references/react-prefer-required-props.md)

- **No `React.FC`** — use an explicit prop type and `): ReactElement` return annotation.
  [react-avoid-react-fc](references/react-avoid-react-fc.md)

- **Optional “render nothing”** — prefer `return undefined` and `ReactElement | undefined` over
  `return null` / `ReactElement | null` so lint stays clean without disables.
  [react-optional-render-nothing](references/react-optional-render-nothing.md)

- **`ReactElement` is ambient** — do not import it; `ReactNode` must be imported.
  [react-common-prop-types](references/react-common-prop-types.md)

- **Plain function declarations** — use `function handleClick(): void {}` over arrow assignments for
  handlers and components.
  [react-function-declaration-style](references/react-function-declaration-style.md)

- **Destructure component props in the signature** — prefer
  `function Component({ a, b }: ComponentProps): ReactElement` over
  `function Component(props: ComponentProps): ReactElement` plus
  `const { a, b } = props;`.
  rules

- **`useEffect` comment** — add a `//` comment on the line directly above every `useEffect`
  explaining why it exists.
  [react-useeffect-rules](references/react-useeffect-rules.md)

- **Complete dependency arrays** — do not suppress the exhaustive-deps lint rule; restructure
  instead.
  [react-useeffect-rules](references/react-useeffect-rules.md)

- **No manual memoization** — React Compiler handles optimization. Do **not** add `useCallback` or
  `useMemo`; use plain `function handle(): void {}` handlers and plain `const` / locals for derived
  values. Avoid `memo` unless there is a documented exception (perf trace, third-party API requiring
  stable refs, etc.).
  [react-compiler](references/react-compiler.md)

- **Compiler-safe control flow in hooks/components** — do not use `try/finally` in compiled hooks;
  avoid ternaries / `&&` / optional chaining and similar “value blocks” _inside_ a `try` (hoist
  logic above `try`, or move async work to a plain sibling `.ts` helper). Dev build uses
  `panicThreshold: "all_errors"` — these mistakes fail Vite, not only lint.
  [react-compiler-control-flow](references/react-compiler-control-flow.md)

- **Strong preference: one component per `.tsx` file** — colocate tests in the same directory.
  [react-file-and-import-conventions](references/react-file-and-import-conventions.md)

- **Paired hook for non-display logic (strong preference)** — treat **`use<SameNameAsComponent>`**
  in **`use<SameNameAsComponent>.ts`** as the default home for substantial state, effects, refs, or
  routing tied to that default-export module (`FooBar` ↔ `useFooBar`). Move behavior hooks,
  handlers/functions, domain-derived consts, hrefs, permission booleans, and async orchestration
  into the paired hook; keep JSX, icons, `className`/Tailwind strings, and style/layout constants in
  the component. Sibling shell and body files each get their own hook when both carry logic; thin
  shells may stay inline until logic grows. **One hook call per component** — fold shell chrome and
  banner/badge placement derivations into the existing `use<Component>` return instead of a second
  colocated hook the `.tsx` must call (see `ToolsTabsSlidePanel` / `useToolsTabsSlidePanel`).
  Components that render copy keep `t(...)` beside the JSX; hooks may call locale hooks for
  non-display needs like `lang`, but never accept, return, or prop-drill `t`. Keep the `.tsx`
  presentation-focused (hook once, destructure, render).
  [react-paired-hook-for-non-display-logic](references/react-paired-hook-for-non-display-logic.md)

- **Direct imports** — no barrel `index.ts` re-exports; import from source files.
  [react-file-and-import-conventions](references/react-file-and-import-conventions.md)

## Output format

Write code changes directly. After edits, output a brief bullet list of which conventions were
applied and why, and which validation commands were run. For question-answering: concise prose with
inline code.

## Error handling

- If `npm run lint` or `npx tsc -b .` fails after changes, report verbatim and fix before declaring
  success.
- If the task would require memoization, flag it and ask the user to confirm before adding it.

## Validation

```bash
npm run lint
```

## Evaluations (I/O examples)

**Input:** "Remove the useCallback from this event handler in `MyComponent.tsx`"
**Expected:** Reads the file, removes `useCallback` wrapper, converts to `function handle(): void
{}`, runs lint and tsc, reports what changed and that React Compiler handles memoization.

**Input:** "Add a useEffect that syncs `selectedId` to localStorage"
**Expected:** Adds `useEffect` with a `//` comment above it explaining why, includes `selectedId` in
the dep array, runs lint. Does not add `useCallback`.

**Input:** "Write tests for `useMyHook.ts`"
**Expected:** Loads `vitest-test-hook-best-practices` skill and proceeds per that skill's guidance,
not this one.

## Skill handoffs

- Hook tests → load `vitest-test-hook-best-practices`.
- TypeScript-only files → load `paniolo-typescript-best-practices` instead.

## Do Not

- Do not use for general TypeScript-only utilities with no React imports.
  Load `paniolo-typescript-best-practices` instead.
- Do not add `useCallback` or `useMemo` — React Compiler handles memoization.
- Do not use `try/finally` inside compiled hooks.
  Hoist logic above `try` or move to a sibling `.ts` helper.
- Do not use `React.FC` — use an explicit prop type with `): ReactElement` return annotation.
- Do not write hook tests with this skill — load `vitest-test-hook-best-practices` instead.

## References

- [react](references/react.md) — full reference
- Repo-wide rules: rules
- [zustand-best-practices/SKILL.md](../paniolo-zustand-best-practices/SKILL.md)
- [paniolo-typescript-best-practices/SKILL.md](../paniolo-typescript-best-practices/SKILL.md)
