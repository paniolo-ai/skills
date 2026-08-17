---
name: paniolo-zustand-best-practices
description: |
  Zustand state management patterns — store creation, selectors, Immer middleware, async actions with loading states, devtools, persist, and testing. Use when authoring or editing Zustand stores (use*Store files) or components that subscribe to stores. Do NOT use for React component structure or TypeScript-only utilities.
license: MIT
metadata:
  version: 0.1.0
tags:
- zustand
- state
references:
- 'wiki: sharp-shooter-wiki:async-actions-with-loading-states → references/async-actions-with-loading-states.md'
- 'wiki: sharp-shooter-wiki:basic-selector-pattern → references/basic-selector-pattern.md'
- 'wiki: sharp-shooter-wiki:devtools → references/devtools.md'
- 'wiki: sharp-shooter-wiki:immer-middleware → references/immer-middleware.md'
- 'wiki: sharp-shooter-wiki:memoized-selectors → references/memoized-selectors.md'
- 'wiki: sharp-shooter-wiki:persist-middleware → references/persist-middleware.md'
- 'wiki: sharp-shooter-wiki:selector-factory-pattern → references/selector-factory-pattern.md'
- 'wiki: sharp-shooter-wiki:store-composition → references/store-composition.md'
- 'wiki: sharp-shooter-wiki:testing-zustand-unit-testing → references/testing-zustand-unit-testing.md'
- 'wiki: sharp-shooter-wiki:zustand → references/zustand.md'
---

**Requires:** file-read, terminal (linting/testing). No network access needed.

## Preconditions

- Read the store file before editing.
- Check rules for repo-wide constraints.

## Defaults (proceed without asking)

- Apply all key rules below; edit the file already open or mentioned.
- **Always ask:** which store file if not specified and cannot be inferred.

## Key rules

- **Always use selectors** — never subscribe to the whole store; select only the fields needed.
  [basic-selector-pattern](references/basic-selector-pattern.md)

- **`useShallow` for derived arrays/objects** — prevents unnecessary re-renders when a selector
  returns a new reference.
  [memoized-selectors](references/memoized-selectors.md)

- **Named selector constants** — define selectors outside components for reuse and stability.
  [selector-factory-pattern](references/selector-factory-pattern.md)

- **One store per domain** — split by concern; do not grow one monolithic store.
  [store-composition](references/store-composition.md)

- **Track `isLoading` and `error`** for every async action.
  [async-actions-with-loading-states](references/async-actions-with-loading-states.md)

- **Use Immer** for deeply nested state updates.
  [immer-middleware](references/immer-middleware.md)

- **Wrap with `devtools`** in all stores; use `partialize` when adding `persist`.
  [devtools](references/devtools.md) and [persist-middleware](references/persist-middleware.md)

- **Never mock the store in tests** — reset with `useStore.setState(initialState)` in `beforeEach`.
  [testing-zustand-unit-testing](references/testing-zustand-unit-testing.md)

## Output format

Write code changes directly. After edits, output a brief bullet list of which patterns were applied
and which validation commands were run.

## Error handling

If the project's lint script fails after changes, report verbatim and fix before declaring success.

## Validation

```bash
<your lint command>
```

## References

- [zustand](references/zustand.md)
- [basic-selector-pattern](references/basic-selector-pattern.md)
- [memoized-selectors](references/memoized-selectors.md)
- [immer-middleware](references/immer-middleware.md)
- [async-actions-with-loading-states](references/async-actions-with-loading-states.md)
- [devtools](references/devtools.md)
- [persist-middleware](references/persist-middleware.md)
- [testing-zustand-unit-testing](references/testing-zustand-unit-testing.md)
- Related skills: [`react-best-practices`](../paniolo-react-best-practices/SKILL.md),
  [`paniolo-typescript-best-practices`](../paniolo-typescript-best-practices/SKILL.md)

## Do Not

- Do not subscribe to the whole store — always select only needed fields.
- Do not mock the store in tests — reset with `useStore.setState(initialState)` in `beforeEach`.
- Do not use `useCallback`/`useMemo` on selectors — React Compiler handles this.
- Do not put UI or component logic inside the store — stores hold state and actions only.
- Do not use for React component structure or TypeScript-only utilities.
  Load the relevant skill instead.
