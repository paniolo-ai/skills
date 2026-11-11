---
name: paniolo-source-refactoring
description: |
  Refactor existing code and break large consolidated files or helper aggregates into single-function modules. Covers single-export default exports, JSDoc preservation, import updates, test colocation, and avoiding leftover moved comments or barrel re-exports. Use when refactoring, modularizing, or decomposing multi-export files. Do NOT use for authoring new files from scratch.
license: MIT
metadata:
  version: 0.2.0
tags:
- refactoring
- files
references:
- references/avoid-multi-function-helpersts-modules.md
- references/file-organization.md
- references/vitest-shared-mock-helper-infrastructure.md
---

**Requires:** file-read, terminal (linting/testing). No network access needed.

# Source Refactoring Skill

## Use When

Use this skill when:

- Splitting large files into focused modules or components.
- Decomposing multi-export aggregate helper files into single-function modules.
- Moving logic into new files while preserving behavior, JSDoc, and tests.

Execution workflow:

1. Identify all call sites and import sites before touching code.
2. Extract one coherent symbol or function at a time with minimal behavior change.
3. Keep exports and imports aligned with repository conventions and avoid barrel files.
4. Move or create colocated tests for extracted logic.
5. Validate with targeted tests first, then run lint.

## Key Rules

### 1. Default Export for Single-Symbol Modules

When a new file contains a single exported function or component, make it the default export on the declaration:

```typescript
// ✅ GOOD: Single exported function as default export
/**
 * @param value - The value to check
 * @returns True if value is a record
 */
export default function isRecord(value: unknown): value is Record<string, unknown> {
	return typeof value === "object" && value !== null && !Array.isArray(value);
}
```

If the file exports multiple tightly related symbols (such as component props and a component, or related type definitions), use named exports or a combination:

```typescript
// ✅ GOOD: Multiple related exports
export type FooProps = { ... };
export function Foo() { ... }
```

### 2. Split Multi-Export Helper Aggregates

Do not allow `react/`, `api/`, or `shared/` to grow aggregate `*Helpers.ts` files with several named exports. Extract each function into its own camelCase default-export module and colocated test.

### 3. No Barrel Re-Exports

Do not create an `index.ts` file that merely re-exports the split files. Call sites must import directly from the specific module path.

### 4. Absolute Imports for Shared Test Helpers

When extracting test helpers, utilities, or shared fixtures imported from multiple locations, use absolute paths (`@/` or package aliases) rather than fragile relative paths (`../../..`). Test helper files must use the `.test-util.ts[x]` suffix.

### 5. Avoid Module-Level Side Effects in Test Helpers

Test helper modules must export callable setup functions rather than auto-executing at import time. Use `vi.hoisted()` for shared mock state.

### 6. Preserve JSDoc

Always copy JSDoc comments from the original file to the new file verbatim. Ensure parameter and return descriptions remain accurate. Do not add redundant TypeScript types inside JSDoc comments.

### 7. Colocate Tests

When moving a symbol that has existing tests:

1. Create `NewFile.test.ts` next to `NewFile.ts`.
2. Move the relevant test cases from the old test file to the new one.
3. Update imports in both test files.
4. Run targeted tests to confirm green status before removing the original file.

### 8. Strict Prohibition on Leftover "Moved To" Comments

Never leave behind refactoring notes such as `// symbol moved to ./path`. These lines are forbidden in the codebase and must be removed before committing:

```bash
git grep -nE "^\s*//.*\bmoved to\b.*$" || true
```

If a migration note is useful for reviewers, place it in the commit message or PR description instead.

### 9. Remove Consolidated File When Empty

Once all symbols and tests have migrated, delete the original file. Do not leave behind empty or comment-only files.

## Step-by-Step Refactoring Process

1. **Identify the target**: Select the function, component, or helper to extract.
2. **Scan usages**: Search the codebase (`grep` or symbol search) for all import sites.
3. **Create the module**: Create `NewSymbol.ts` or `NewSymbol.tsx` with `export default function NewSymbol()`.
4. **Transfer logic and JSDoc**: Copy function implementation and its JSDoc comments.
5. **Colocate tests**: Create `NewSymbol.test.ts` and transfer the corresponding tests.
6. **Update imports**: Update all call sites from named imports to default imports.
7. **Clean up**: Remove leftover refactoring comments; remove the original file if fully migrated.
8. **Verify**: Run targeted unit tests followed by project linting.

## Validation

```bash
<your unit-test command> path/to/changed.test.ts
<your lint command>
```

## Evaluations (I/O Examples)

**Input:** "Split `shared/src/utils/helpers.ts` into separate files"
**Expected:** Agent reads the file, identifies each exported symbol, creates one file per symbol named after the symbol with default export, updates all import sites, colocates tests, removes leftover moved comments, removes original file, and runs lint and tests.

**Input:** "Extract `formatDate` helper to its own file"
**Expected:** Agent creates `formatDate.ts` next to the source, moves function and JSDoc as `export default function formatDate`, updates import sites, runs validation, and creates no barrel re-export.

## Do Not

- Do not create barrel `index.ts` re-exports.
- Do not drop or abbreviate JSDoc when moving symbols.
- Do not leave `// moved to` comments in source code.
- Do not create new files from scratch without an existing symbol to split or refactor.
- Do not add broad lint or type suppressions during refactoring.

## References

- [file-organization](../paniolo-file-organization/SKILL.md)
- [vitest-test-best-practices](../paniolo-vitest-test-best-practices/SKILL.md)

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
