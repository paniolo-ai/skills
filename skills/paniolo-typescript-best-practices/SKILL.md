---
name: paniolo-typescript-best-practices
description: TypeScript conventions for any repo — no `any`, explicit return types, JSDoc, exactOptionalPropertyTypes, ambient types, import style. Use when authoring or editing `.ts`/`.tsx`, resolving strictness errors, or fixing type-related lint failures. Do NOT use for React-specific typing (see paniolo-react-best-practices) or JSDoc-only changes (see paniolo-code-comment-best-practices).
license: MIT
metadata:
  version: 0.1.0
  homepage: https://github.com/paniolo-ai/skills
tags:
- typescript
references:
- 'wiki: sharp-shooter-wiki:typescript-ambient-types → references/typescript-ambient-types.md'
- 'wiki: sharp-shooter-wiki:typescript-avoid-any → references/typescript-avoid-any.md'
- 'wiki: sharp-shooter-wiki:typescript-avoiding-redundant-type-assertions → references/typescript-avoiding-redundant-type-assertions.md'
- 'wiki: sharp-shooter-wiki:typescript-consistent-type-imports-type-only-imports → references/typescript-consistent-type-imports-type-only-imports.md'
- 'wiki: sharp-shooter-wiki:typescript-default-export-functions → references/typescript-default-export-functions.md'
- 'wiki: sharp-shooter-wiki:typescript-exactoptionalpropertytypes-handling → references/typescript-exactoptionalpropertytypes-handling.md'
- 'wiki: sharp-shooter-wiki:typescript-explicit-return-types → references/typescript-explicit-return-types.md'
- 'wiki: sharp-shooter-wiki:typescript-function-parameters → references/typescript-function-parameters.md'
- 'wiki: sharp-shooter-wiki:typescript-strict-null-checks → references/typescript-strict-null-checks.md'
- 'wiki: sharp-shooter-wiki:typescript-type-vs-interface → references/typescript-type-vs-interface.md'
- 'wiki: sharp-shooter-wiki:typescript-typescript → references/typescript-typescript.md'
- 'wiki: sharp-shooter-wiki:vitest-colocate-make-fixtures → references/vitest-colocate-make-fixtures.md'
- 'wiki: sharp-shooter-wiki:vitest-forcecast-vs-make-fixtures → references/vitest-forcecast-vs-make-fixtures.md'
---

**Requires:** file-read, terminal (linting/testing). No network access needed.

**Full reference:** [typescript-typescript](references/typescript-typescript.md)

- Editing any `.ts` or `.tsx` file.
- Resolving TypeScript strictness or lint errors related to typing quality.

## Execution workflow

1. Apply relevant rules below (each links to the full doc for examples).
2. Keep changes minimal and local to the problem area.
3. Run `npm run lint` after meaningful TS changes.

## Key rules

- **No `any`** — use `unknown` + type guards instead.
  [typescript-avoid-any](references/typescript-avoid-any.md)

- **Prefer `type` over `interface`** — for object shapes, unions, and function signatures.
  [typescript-type-vs-interface](references/typescript-type-vs-interface.md)

- **Explicit return types** — always annotate function return types.
  [typescript-explicit-return-types](references/typescript-explicit-return-types.md)

- **JSDoc on new/changed functions** — concise, purpose-focused; no `{...}` types or
  `@returns {Foo}` in `.ts` / `.tsx` (prose-only `@param` / `@returns`).
  [→ code-comment-best-practices skill](../paniolo-code-comment-best-practices/SKILL.md)
- **Keep JSDoc in sync** — if you change params, props, or the behavior described by existing JSDoc,
  update the JSDoc in the same edit.

- **Ambient types** — `ReactElement` is ambient (no import needed); `ReactNode` must be imported.
  [typescript-ambient-types](references/typescript-ambient-types.md)

- **Destructure params in signature** — prefer destructured parameters in the function signature
  (e.g., `function foo({ a, b }: Options)`) over a single params object with internal destructuring.
  This satisfies the max-parameters linter rule while keeping code readable.
  [typescript-function-parameters](references/typescript-function-parameters.md)

- **`exactOptionalPropertyTypes`** — use conditional spread `...(x === undefined ? {} : { x })` to
  thread optional props; never pass `T | undefined` where `T` is expected.
  [typescript-exactoptionalpropertytypes-handling](references/typescript-exactoptionalpropertytypes-handling.md)

- **No redundant casts** — prefer an explicit type annotation on the receiving variable over an
  inline `as` cast.
  [typescript-avoiding-redundant-type-assertions](references/typescript-avoiding-redundant-type-assertions.md)

- **`Set` from optional arrays** — `new Set(optionalArray)` needs no `?? []` fallback.
  [typescript-strict-null-checks](references/typescript-strict-null-checks.md)

- **Type-only imports** — `import type { }` for all-type imports; inline `type` keyword for mixed
  imports.
  [typescript-consistent-type-imports-type-only-imports](references/typescript-consistent-type-imports-type-only-imports.md)

- **Default export functions** — use inline declarations: `export default function foo()` or
  `export default async function foo()`. Do not declare the function and then add a trailing
  `export default foo;` statement. This keeps the declaration and its export co-located.
  [typescript-default-export-functions](references/typescript-default-export-functions.md)

- **No `forceCast` outside tests** — `forceCast` (in `lib/test-utils/`) is a test-only escape hatch.
  In tests, prefer shared `make*` helpers with `Partial` overrides for typed rows; colocate
  feature-specific helpers with the feature folder; see
  [vitest-forcecast-vs-make-fixtures](references/vitest-forcecast-vs-make-fixtures.md) and
  [vitest-colocate-make-fixtures](references/vitest-colocate-make-fixtures.md).
  In production code, first ask whether a generic type parameter can simply be removed or replaced
  with a concrete type so no cast is needed at all. If a cast is truly unavoidable, use
  `as unknown as TheType` (no `any`) and justify it with a comment; never use lint-disable comments
  as a substitute for a proper type fix.

## Defaults (proceed without asking)

- Strict mode and `exactOptionalPropertyTypes: true` are always on.
- `no-null` lint rule is active; use `oxlint-disable-next-line no-null` only when clearing nullable
  FK columns.

## Output format

- Inline code edits only — no standalone files unless asked.
- After edits, summarize which rules were applied; note any unavoidable tradeoffs (e.g., a temporary
  `as` assertion with justification).

## Validation

```bash
npm run lint       # lint (always from project root)
npm run test:unit  # when behavior is non-trivial
```

## Skill handoffs

- Lint failures → also load `paniolo-lint-error-resolution`.
- React-focused edits → also load `paniolo-react-best-practices`.
- JSDoc-only changes → load `paniolo-code-comment-best-practices`.

## Do Not

- Do not use for React-specific typing patterns — load `paniolo-react-best-practices` instead.
- Do not use for JSDoc-only changes — load `paniolo-code-comment-best-practices` instead.
- Do not use `any` — use `unknown` with type guards instead.
- Do not add inline `as` casts without a justification comment.
  Prefer `as unknown as TheType` over `as any`.
- Do not suppress lint rules without explicit justification.

## References

- Full reference: [typescript-typescript](references/typescript-typescript.md)
- [paniolo-react-best-practices/SKILL.md](../paniolo-react-best-practices/SKILL.md)
- [paniolo-code-comment-best-practices/SKILL.md](../paniolo-code-comment-best-practices/SKILL.md)
- [paniolo-lint-error-resolution/SKILL.md](../paniolo-lint-error-resolution/SKILL.md)