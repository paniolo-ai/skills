---
name: paniolo-typescript-lint
description: |
  Pre-authoring checklist, workflows, and resolution lookup for oxlint, ESLint, and tsc errors in strict TypeScript codebases. Use when authoring TypeScript to pass lint or resolving TypeScript lint errors. Do NOT use for Markdown (see paniolo-markdown-lint-best-practices) or Rust clippy.
license: MIT
metadata:
  version: 0.1.0
tags:
- typescript
- lint
- oxlint
- eslint
- tsc
user-invocable: true
references:
- references/typescript-all-if-else-branches-contain-same-code-shared-error-check.md
- references/typescript-isolateddeclarations-explicit-type-annotations-on-exports.md
- references/typescript-lint.md
- references/typescript-no-unsafe-assignment-no-unsafe-call-no-unsafe-member-access-dyna.md
- references/typescript-no-unsafe-type-assertion-request-validation.md
- references/typescript-optional-method-chain-use-typed-query-wrapper.md
- references/typescript-quick-lookup.md
- references/typescript-require-useeffect-comment-comment-before-useeffect.md
---

**Requires:** file-read, terminal (oxlint, eslint, tsc).

# TypeScript Lint

## Use When

- Writing or refactoring TypeScript in any workspace repository.
- Resolving lint or type errors from `oxlint`, `ESLint`, or `tsc`.
- Checking code against strict TypeScript conventions before committing.

---

## Pre-Authoring Checklist

Apply these conventions while authoring to pass lint on the first try:

### Types
- **No `any`:** Use `unknown` with type narrowing, type guards, or schema validation.
- **Explicit return types:** Required on exported functions (`isolatedDeclarations`).
- **Type-only imports:** Use `import type { ... }` for types and interfaces.

### Imports
- **No barrel files:** Import directly from source modules, not re-export barrels (`oxc/no-barrel-file`).

### Control Flow & Style
- **Curly braces:** Always use braces on all control flow statements (`curly`).
- **No magic numbers:** Declare named constants (`no-magic-numbers`).
- **Standard builtins:** Use `Number.parseInt()` / `Number.isNaN()`, `.toSorted()`, and `catch (error: unknown)`.
- **Naming:** Variable names must be ≥ 2 characters; place the positive condition branch first.

### Tests
- Arrange-Act-Assert (AAA) comments in test blocks.
- No lint disable annotations in test files.

---

## Lint Error Resolution Workflow

When resolving existing lint failures:

1. **Get exact message:** Read the file containing the error and locate the exact line and rule ID. Do not guess root causes.
2. **Determine lint command:** Read `package.json` `scripts` and use the repo's specific lint entrypoint — typically `lint:ts`, falling back to `lint`.
   - Never substitute bare `npx eslint` for the repository's configured script.
   - Do not invoke unlisted script names.
3. **Apply minimal fix:** Fix the root cause directly using the conventions above.
4. **Re-run lint:** Verify with the repository's lint script and confirm 0 errors before proceeding.

---

## Quick Lookup — Common Rules

- `id-length`, `no-magic-numbers`, `no-negated-condition`, `prefer-number-properties`, `curly`, `consistent-type-imports`, `unicorn/no-array-sort` → [typescript-quick-lookup](references/typescript-quick-lookup.md)
- `no-unsafe-type-assertion` (API routes) → [Request validation](references/typescript-no-unsafe-type-assertion-request-validation.md)
- `no-unsafe-assignment` (dynamic DB tables) → [Dynamic tables](references/typescript-no-unsafe-assignment-no-unsafe-call-no-unsafe-member-access-dyna.md)
- "All if-else branches contain same code" → [Shared error handling](references/typescript-all-if-else-branches-contain-same-code-shared-error-check.md)
- Query-builder optional chain → [Typed query wrapper](references/typescript-optional-method-chain-use-typed-query-wrapper.md)
- `require-useeffect-comment` → [useEffect comments](references/typescript-require-useeffect-comment-comment-before-useeffect.md)
- `--isolatedDeclarations` / Effect schemas → [isolatedDeclarations](references/typescript-isolateddeclarations-explicit-type-annotations-on-exports.md)

---

## When to Disable Rules (Rare)

Disables require a `-- reason` explanation and must be strictly exceptional:

```typescript
// oxlint-disable-next-line typescript/no-unsafe-assignment -- TODO: type upstream third-party payload
const legacy = thirdParty.getData();
```

---

## References

- Full reference: [typescript-lint](references/typescript-lint.md)
- [paniolo-typescript-best-practices](../paniolo-typescript-best-practices/SKILL.md) — language and type system conventions
- [paniolo-react-best-practices](../paniolo-react-best-practices/SKILL.md) — React component and hook conventions

## Do Not

- Do not use `any` to bypass typecheck or lint errors.
- Do not suppress rules with disable comments without an explicit `-- reason`.
- Do not substitute bare `npx eslint` when the repository defines a package script.
- Do not expand scope beyond requested changes when resolving lint failures.
- Do not use this skill for Markdown linting (see `paniolo-markdown-lint-best-practices`) or Rust (see `cargo clippy`).

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
