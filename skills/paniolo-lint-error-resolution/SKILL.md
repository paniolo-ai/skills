---
name: paniolo-lint-error-resolution
description: |
  Guide for resolving lint errors in strict TypeScript environments — oxlint, ESLint, and tsc. Use when encountering lint errors in files being edited. Load the repo-specific lint supplement when fixing a workspace app repo's code.
license: MIT
metadata:
  version: 0.1.0
tags:
- lint
- typescript
references:
- 'wiki: sharp-shooter-wiki:typescript-all-if-else-branches-contain-same-code-supabase-error-check → references/typescript-all-if-else-branches-contain-same-code-supabase-error-check.md'
- 'wiki: sharp-shooter-wiki:typescript-isolateddeclarations-explicit-type-annotations-on-exports → references/typescript-isolateddeclarations-explicit-type-annotations-on-exports.md'
- 'wiki: sharp-shooter-wiki:typescript-lint → references/typescript-lint.md'
- 'wiki: sharp-shooter-wiki:typescript-no-unsafe-assignment-no-unsafe-call-no-unsafe-member-access-dyna → references/typescript-no-unsafe-assignment-no-unsafe-call-no-unsafe-member-access-dyna.md'
- 'wiki: sharp-shooter-wiki:typescript-no-unsafe-type-assertion-request-validation → references/typescript-no-unsafe-type-assertion-request-validation.md'
- 'wiki: sharp-shooter-wiki:typescript-quick-lookup → references/typescript-quick-lookup.md'
- 'wiki: sharp-shooter-wiki:typescript-require-useeffect-comment-comment-before-useeffect → references/typescript-require-useeffect-comment-comment-before-useeffect.md'
- 'wiki: sharp-shooter-wiki:typescript-supabasefromlike-optional-method-chain-use-callselect → references/typescript-supabasefromlike-optional-method-chain-use-callselect.md'
---

**Requires:** file-read, terminal (linter). No network access needed.

## When invoked

**Preconditions:**

- Read the file containing the error before attempting a fix.
- Obtain the exact error message and line number — do not guess the root cause.
- Load the repo supplement when errors involve React, API handlers, or Supabase.

**Output format:**

- Apply the fix directly. One sentence on root cause after the change.
- Re-run the repo's lint entrypoint and report the result.

**Lint command policy:**

| Repo     | Entrypoint         |
| -------- | ------------------ |
| harness  | `pnpm run lint:ts` |
| paniolo  | `pnpm run lint`    |
| app repo | `pnpm run lint`    |

Do not substitute bare `npx eslint` for the repo lint script.

**Error handling:**

- If the pattern is not in the lookup tables, load the catalog or repo supplement (links below).
- Rule disables need a `-- reason` annotation and must be rare.

## Quick lookup — shared errors

**Catalog:**
[typescript-quick-lookup](references/typescript-quick-lookup.md)

- `id-length`, `no-magic-numbers`, `no-negated-condition`, `prefer-number-properties`,
  `curly`, `consistent-type-imports`, `unicorn/no-array-sort` → catalog quick lookup
- `require-own-file-for-long-function`, `require-aaa-pattern` → catalog custom ESLint rules

## Quick lookup — React / API / Supabase app repos

- `no-unsafe-type-assertion` (API) →
  [Request validation](references/typescript-no-unsafe-type-assertion-request-validation.md)
- `no-unsafe-assignment` / dynamic Supabase tables →
  [Dynamic tables](references/typescript-no-unsafe-assignment-no-unsafe-call-no-unsafe-member-access-dyna.md)
- "All if-else branches same code" (Supabase) →
  [Supabase errors](references/typescript-all-if-else-branches-contain-same-code-supabase-error-check.md)
- `SupabaseFromLike` optional method chain →
  [callSelect](references/typescript-supabasefromlike-optional-method-chain-use-callselect.md)
- `require-useeffect-comment` →
  [useEffect](references/typescript-require-useeffect-comment-comment-before-useeffect.md)
- `--isolatedDeclarations` / Effect schema exports →
  [isolatedDeclarations](references/typescript-isolateddeclarations-explicit-type-annotations-on-exports.md)

## Quick lookup — Paniolo scan

**Supplement:**
paniolo

## When to disable rules (rare)

```typescript
// oxlint-disable-next-line typescript/no-unsafe-assignment -- TODO: type upstream payload
const legacy = thirdParty.getData();
```

## References

- **Harness catalog:**
  [typescript-lint](references/typescript-lint.md)
- **Paniolo:**
  paniolo
- **Project rules:** rules

## Do not

- Do not suppress rules without justification and `-- reason`.
- Do not expand scope beyond the requested task.