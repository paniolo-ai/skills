---
name: paniolo-react-best-practices
description: |
  React 18+ conventions for this project — ReactElement ambient type, useEffect comment rule, plain function declarations, prop and file/import conventions, paired hooks. Use when authoring or editing any React component, hook, or page. Do NOT use for general TypeScript-only utilities with no React imports — load paniolo-typescript-best-practices instead. For React Compiler constraints (no useCallback/useMemo, no render side effects), load paniolo-react-compiler.
license: MIT
metadata:
  version: 0.2.0
tags:
- react
references:
- 'wiki: sharp-shooter-wiki:react → references/react.md'
- 'wiki: sharp-shooter-wiki:react-avoid-react-fc → references/react-avoid-react-fc.md'
- 'wiki: sharp-shooter-wiki:react-common-prop-types → references/react-common-prop-types.md'
- 'wiki: sharp-shooter-wiki:react-file-and-import-conventions → references/react-file-and-import-conventions.md'
- 'wiki: sharp-shooter-wiki:react-function-declaration-style → references/react-function-declaration-style.md'
- 'wiki: sharp-shooter-wiki:react-optional-render-nothing → references/react-optional-render-nothing.md'
- 'wiki: sharp-shooter-wiki:react-paired-hook-for-non-display-logic → references/react-paired-hook-for-non-display-logic.md'
- 'wiki: sharp-shooter-wiki:react-prefer-required-props → references/react-prefer-required-props.md'
- 'wiki: sharp-shooter-wiki:react-useeffect-rules → references/react-useeffect-rules.md'
---

**Requires:** file-read, terminal (linting/testing). No network access needed.

**Full reference:** [react](references/react.md)

**Companion skills (load on demand):**

- **React Compiler enabled in this repo?** (check `vite.config.ts` or `package.json` for
  `babel-plugin-react-compiler`) → also load
  [react-compiler/SKILL.md](../paniolo-react-compiler/SKILL.md). It forbids `useCallback` /
  `useMemo` and render-phase side effects. This skill is compiler-independent and deliberately says
  nothing about memoization.
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

- **Memoization is out of scope here** — whether `useCallback` / `useMemo` are appropriate depends
  on whether the repo enables the React Compiler. Load
  [react-compiler/SKILL.md](../paniolo-react-compiler/SKILL.md) when it does; that skill forbids
  them outright.

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
- If the task turns on whether memoization is allowed, stop and load `paniolo-react-compiler` (or
  confirm the repo does not enable the compiler) before deciding.

## Validation

```bash
npm run lint
```

## Evaluations (I/O examples)

**Input:** "Remove the useCallback from this event handler in `MyComponent.tsx`"
**Expected:** Loads `paniolo-react-compiler` (the rule lives there), then removes the wrapper and
converts to `function handle(): void {}` per this skill's declaration-style rule. Runs lint and tsc.

**Input:** "Add a useEffect that syncs `selectedId` to localStorage"
**Expected:** Adds `useEffect` with a `//` comment above it explaining why, includes `selectedId` in
the dep array, runs lint.

**Input:** "Write tests for `useMyHook.ts`"
**Expected:** Loads `vitest-test-hook-best-practices` skill and proceeds per that skill's guidance,
not this one.

## Skill handoffs

- React Compiler constraints (memoization, render purity, compiled-hook control flow) → load
  `paniolo-react-compiler`.
- Hook tests → load `vitest-test-hook-best-practices`.
- TypeScript-only files → load `paniolo-typescript-best-practices` instead.

## Do Not

- Do not use for general TypeScript-only utilities with no React imports.
  Load `paniolo-typescript-best-practices` instead.
- Do not rule on `useCallback` / `useMemo` from this skill — that depends on whether the repo
  enables the React Compiler. Load `paniolo-react-compiler`.
- Do not use `React.FC` — use an explicit prop type with `): ReactElement` return annotation.
- Do not write hook tests with this skill — load `vitest-test-hook-best-practices` instead.

## References

- [react](references/react.md) — full reference
- Repo-wide rules: rules
- [paniolo-react-compiler/SKILL.md](../paniolo-react-compiler/SKILL.md)
- [zustand-best-practices/SKILL.md](../paniolo-zustand-best-practices/SKILL.md)
- [paniolo-typescript-best-practices/SKILL.md](../paniolo-typescript-best-practices/SKILL.md)