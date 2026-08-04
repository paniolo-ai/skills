---
name: paniolo-typescript-lint-best-practices
description: |
  Pre-authoring checklist, workflow, and common-error lookup for writing lint-clean TypeScript. Use when writing new code to pass lint on the first try. Load the repo-specific supplement when editing a workspace app repo.
license: MIT
metadata:
  version: 0.1.0
tags:
- typescript
- lint
references:
- 'wiki: sharp-shooter-wiki:typescript-lint → references/typescript-lint.md'
---

**Requires:** file-read, terminal (linting).

**Harness catalog:**
[typescript-lint](references/typescript-lint.md)

**Repo supplements:**

- Paniolo scan:
  paniolo

# TypeScript Lint Best Practices

## Use When

- Writing or refactoring TypeScript in a workspace repo.
- Trying to pass lint on the first try.
- Prompting an agent to write lint-clean TypeScript.

## Do Not Use When

- The primary task is fixing existing lint failures — use
  [lint-error-resolution](../paniolo-lint-error-resolution/SKILL.md) instead.

---

## Preconditions

- Read `AGENTS.md` and rules for the
  repo you are editing.
- Load the **repo supplement** above when working in an app repo.
- Read the file being changed and one nearby file that already follows the local pattern.

---

## Pre-Authoring Checklist (shared)

Apply before writing — see the catalog doc for the full lookup table.

### Types

- No `any` — use `unknown` + narrow or schema validation.
- Explicit return types on exported functions.
- `import type` for type-only imports.

### Imports

- No barrel files — import from source modules (`oxc/no-barrel-file`).

### Style

- Braces on all control flow (`curly`).
- Named constants for magic numbers.
- `Number.parseInt()` / `Number.isNaN()`, `.toSorted()`, `catch (error: unknown)`.
- Variable names ≥ 2 characters; positive branch first.

### Harness `scripts/` only

- AAA markers in tests; no lint-disable in tests; mock interaction assertions only.

### Paniolo scan only

- Load the
  paniolo.

---

## Execution Workflow

1. Load this skill + the repo supplement when applicable.
2. Match surrounding structure, naming, and imports.
3. `pnpm run lint:fix` (if available) → full lint entrypoint → format if needed.

Use the repo's own lint script rather than a remembered one: read its
`package.json` `scripts` and prefer the most specific entrypoint that covers
TypeScript — commonly `lint:ts`, falling back to `lint`.

Do not substitute bare `npx eslint` for the repo's lint script, and do not invoke
a script name you have not seen in that repo's `package.json`.

---

## References

- **Catalog:**
  [typescript-lint](references/typescript-lint.md)
- **Fix failures:**
  [lint-error-resolution](../paniolo-lint-error-resolution/SKILL.md)

## Do Not

- Do not use `any` or unsafe casts without justification.
- Do not import from barrel files.
- Do not stop after `lint:fix` — re-run the full lint entrypoint.