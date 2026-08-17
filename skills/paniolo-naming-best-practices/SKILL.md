---
name: paniolo-naming-best-practices
description: |
  Symbol and file naming conventions for functions, types, variables, and React components. Use when naming a new function, hook, type, file, or variable, or when reviewing whether an existing name is appropriate.
license: MIT
metadata:
  version: 0.1.0
tags:
- naming
references:
- 'wiki: sharp-shooter-wiki:naming → references/naming.md'
- 'wiki: sharp-shooter-wiki:naming-file-naming → references/naming-file-naming.md'
- 'wiki: sharp-shooter-wiki:react-component-naming → references/react-component-naming.md'
- 'wiki: sharp-shooter-wiki:type-and-interface-naming → references/type-and-interface-naming.md'
- 'wiki: sharp-shooter-wiki:typescript-naming-use-vs-compute → references/typescript-naming-use-vs-compute.md'
- 'wiki: sharp-shooter-wiki:typescript-naming-variable-naming → references/typescript-naming-variable-naming.md'
---

**Requires:** file-read, terminal (linting/testing). No network access needed.

**Reference:** [naming](references/naming.md) — code examples
for use\* vs compute\*, type/variable/component naming, file naming table, and success criteria.

## Use When

Use this skill when:

- Naming new files, functions, hooks, types, or variables.
- Renaming symbols during refactors for clarity and convention compliance.

Execution workflow:

1. Choose names based on behavioral intent (fetch, compute, subscribe, run, etc.).
2. Ensure hook names (`use*`) only apply when React hooks are called internally.
3. Keep file and symbol names aligned with nearby project patterns.
4. Recheck references/imports after renames and validate with the project's lint script.

**Output:** call out key symbol/file renames and rationale; note any convention exceptions.

## Function / Hook Prefix Guide

The prefix signals the responsibility of the function. Pick the most precise one.

| Prefix                  | When to use                                                    | Example                                                                 |
| ----------------------- | -------------------------------------------------------------- | ----------------------------------------------------------------------- |
| `use*`                  | React hook — **must** call at least one React hook internally  | `useEventPermissions` → ❌ (no hooks inside); `useActiveEventSync` → ✅ |
| `compute*`              | Pure function that **derives** a value via non-trivial logic   | `computeEventPermissions`, `computeSlidePosition`                       |
| `get*`                  | Simple **retrieval** — property access, map lookup, array find | `getErrorMessage`, `getDbClient`                                       |
| `fetch*`                | **Async** data load (network/DB)                               | `fetchEventBySlug`, `fetchCommunityLibrary`                             |
| `subscribe*`            | Sets up a **realtime / WebSocket** subscription                | `subscribeToCommunityEvent`, `subscribeToPresence`                      |
| `run*`                  | Executes a multi-step **Effect pipeline** or async flow        | `runAction`, `runCommunityAction`                                       |
| `create*`               | **Factory** — returns a new instance or slice                  | `createEventSlice`, `createAuthSlice`                                   |
| `make*`                 | Builds a **data structure** or test double                     | `makeNavigationSliceMock`, `makeUseManageView`                          |
| `build*`                | Incrementally **assembles** a value (builder pattern)          | `buildPathWithLang`                                                     |
| `handle*`               | Implements an **event handler** (not a prop name)              | `handleSubmit`, `handleSelectCommunity`                                 |
| `on*`                   | **Callback prop** passed to a React component                  | `onInviteClick`, `onKickParticipant`                                    |
| `set*`                  | **State setter** (side-effect, void)                           | `setActionState`, `setCurrentEvent`                                     |
| `is*` / `has*` / `can*` | **Boolean predicate** — plain value or derived flag            | `isOwner`, `canManageEvent`, `hasPermission`                            |
| `update*`               | **Mutates** existing state (async or sync)                     | `updateActiveSong`, `updateActiveSlidePosition`                         |
| `refresh*`              | **Re-fetches** already-loaded data                             | `refreshEvent`, `refreshCommunity`                                      |
| `format*`               | Pure **string / display** transformation                       | `formatDate`, `formatDuration`                                          |

[typescript-naming-use-vs-compute](references/typescript-naming-use-vs-compute.md)

## Type / Interface Naming

Suffix patterns: `*Props` (component props), `*Return`/`*Result` (return shapes),
`*Error` (error types), `*Slice`/`*State` (store slices), short PascalCase for generics.

[type-and-interface-naming](references/type-and-interface-naming.md)

## Variable Naming

Booleans: `is*`/`has*`/`can*`. Constants: `UPPER_SNAKE_CASE`. Locals: `camelCase`.
Callbacks: `handle*`. Refs: `camelCase` + `Ref` suffix.

[typescript-naming-variable-naming](references/typescript-naming-variable-naming.md)

## React Component Naming

Components: `PascalCase`, filename matches export. Providers: `*Provider` suffix.
Sub-components: `PascalCase`, descriptive.

[react-component-naming](references/react-component-naming.md)

## File Naming

Components: `PascalCase.tsx`. A **single default-exported function**: `camelCase.ts` (do **not** use
kebab-case). Multi-symbol modules: `kebab-case.ts`. Types-only, **single** exported type:
`PascalCase.type.ts`. Types-only **multiple** exports: `kebab-case.type.ts`.
Tests: `source.test.ts`. Directories: `kebab-case`.

[naming-file-naming](references/naming-file-naming.md)

## Validation Checklist

- [ ] Prefix accurately describes what the function **does** (not what it returns)
- [ ] If prefixed `use*`, calls at least one React hook internally
- [ ] Pure derivations use `compute*` rather than `get*` or `use*`
- [ ] Booleans start with `is*`, `has*`, or `can*`
- [ ] Filename matches the primary exported symbol

## Do Not

- Do not use `use*` prefix unless the function calls at least one React hook internally.
- Do not use `get*` for non-trivial derivations — use `compute*` instead.
- Do not use this skill for file organization or import decisions.
  Load `file-organization` instead.
- Do not rename symbols without updating all import sites and running the project's lint script.

## References

- [file-organization skill](../paniolo-file-organization/SKILL.md) — full file/directory
  naming rules
- [react-best-practices skill](../paniolo-react-best-practices/SKILL.md) — React-specific
  patterns
- [paniolo-typescript-best-practices skill](../paniolo-typescript-best-practices/SKILL.md) — type
  conventions
- rules — canonical project rules

## Skill Handoffs

- Rename work includes moving files/modules → also load `file-organization`.
- Rename work is part of a larger refactor → also load `source-refactoring`.
