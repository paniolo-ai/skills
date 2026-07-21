---
name: paniolo-file-organization
description: |
  File organization, naming conventions, and import patterns (no barrel files, direct imports, kebab-case docs, ESM config). Use when creating new files, modules, refactoring imports, or setting up directory structures.
license: MIT
metadata:
  version: 0.1.0
tags:
- files
- structure
references:
- 'wiki: sharp-shooter-wiki:avoid-multi-function-helpersts-modules → references/avoid-multi-function-helpersts-modules.md'
- 'wiki: sharp-shooter-wiki:code-layout → references/code-layout.md'
- 'wiki: sharp-shooter-wiki:component-files-react-components → references/component-files-react-components.md'
- 'wiki: sharp-shooter-wiki:constants-placement-across-modules → references/constants-placement-across-modules.md'
- 'wiki: sharp-shooter-wiki:file-organization → references/file-organization.md'
- 'wiki: sharp-shooter-wiki:forbidden-feature-folder-names → references/forbidden-feature-folder-names.md'
- 'wiki: sharp-shooter-wiki:import-organization-pattern → references/import-organization-pattern.md'
- 'wiki: sharp-shooter-wiki:naming-conventions → references/naming-conventions.md'
- 'wiki: sharp-shooter-wiki:project-structure-example → references/project-structure-example.md'
- 'wiki: sharp-shooter-wiki:vitest-colocate-make-fixtures → references/vitest-colocate-make-fixtures.md'
---

**Requires:** file-read, terminal (linting/testing). No network access needed.

# File Organization Skill

## Use When

Use this skill when:

- Creating new files/directories or restructuring module layout.
- Updating imports/exports to comply with repo organization rules.

Execution workflow:

1. Follow naming and placement conventions for file type and symbol type.
2. Avoid barrel files and use direct imports.
3. Keep test/docs naming conventions aligned with project rules.
4. Validate with `npm run lint` after meaningful structure/import changes.

Output requirements:

- Summarize structural and import-path changes.
- Note any convention exception that remains and why.

## Key Rules

### 1. No Barrel Files — Always Import Directly

❌ **Never create `index.ts` re-export files:**

```typescript
// BAD: react/src/components/index.ts
export { SongCard } from "./SongCard";
export { UserProfile } from "./UserProfile";
```

✅ **Always import directly from source:**

```typescript
// GOOD
import { SongCard } from "./components/SongCard";
import { UserProfile } from "./components/UserProfile";
```

This applies everywhere: no `export { something } from "./somefile"` re-exports. Also avoid
pass-through re-exports from function/component files. Never create re-export files (e.g.,
`tools-tab-row.type.ts` re-exporting from `tab-config.type`) — delete them and update callers
to import directly from the defining module.

**Linting requirement:** After making any code change (including refactors and imports), run `npm
run lint` from the project root and address any failures before finalizing the change or opening a
PR. If lint fails, do not merge — fix or escalate.

### 2. Explicit Type Imports

```typescript
// ✅ All types — use import type
import type { PopoverProps, PopoverState } from "./popover/types";

// ✅ Mixed imports — type keyword on individual types
import { NativePopover } from "./popover/NativePopover";
import { type PopoverProps } from "./popover/types";

// ❌ Avoid inline type when all imports are types
import { type PopoverProps, type PopoverState } from "./popover/types";
```

### 3. Naming Conventions

→ [naming-conventions](references/naming-conventions.md) covers PascalCase
components, **camelCase** for a **single default-exported function** (app and e2e), kebab-case
multi-symbol files, test colocation, test-util suffix, docs naming, ESM config, and Bun scripts. For
React `.tsx` patterns (one component per file, sibling shells), see the
[component-files-react-components](references/component-files-react-components.md).

### 4. Project Structure

→ [project-structure-example](references/project-structure-example.md) shows `api/src/`,
`react/src/`, `shared/src/`, `docs/`, and `scripts/` layout with feature grouping.
**Key rules:** Feature/domain directories, one main export per file, colocated tests, no `index.ts`
re-exports, no new multi-export `*Helpers.ts` in `react/` / `api/` / `shared/` (see
[avoid-multi-function-helpersts-modules](references/avoid-multi-function-helpersts-modules.md)), max
3–4 nesting levels. When **any code directory** exceeds ~12 direct modules, split into
behavior-named feature subfolders — see
[code-layout](references/code-layout.md) (`pnpm run lint` runs the scan `code-layout` rule in
harness). Never name feature subfolders `helpers/`, `constants/`, `types/`, `utils/`, or other junk
drawers — see [forbidden-feature-folder-names](references/forbidden-feature-folder-names.md). Feature-specific
`make*.test-util.ts` fixtures belong in the feature folder (not generic `test-utils/` unless
cross-cutting) — see [vitest-colocate-make-fixtures](references/vitest-colocate-make-fixtures.md).

### 5. Absolute Paths for Cross-Module Imports

Use absolute paths (`@/`) for test helpers or utilities imported from **multiple different
locations**:

```typescript
// ✅ GOOD: Same absolute path from any caller
import mockUseSlideManagerView from "@/react/event/manage/test-utils/mockUseSlideManagerView.test-util";

// ❌ BAD: Relative paths break when caller depth changes
import mockUseSlideManagerView from "./test-utils/mockUseSlideManagerView.test-util";
```

Path aliases: `@/api/` = `api/src/`, `@/shared/` = `shared/src/`, `@/react/` = `react/src/`

### 6. Import Order Within a File

External packages, then absolute internal imports, then relative imports,
then type-only imports last — see [import-organization-pattern](references/import-organization-pattern.md)
for the full example.

## Validation

```bash
npm run lint                                          # Check all violations
find . -name "index.ts" -not -path "./node_modules"  # Detect barrel files
find . -name "*.cjs" -not -path "./node_modules"     # Detect CommonJS config
```

## References

- Reference guide:
  [file-organization](references/file-organization.md)
- Project rules:
  rules
- TypeScript conventions:
  [../external/paniolo-typescript-best-practices/SKILL.md](../paniolo-typescript-best-practices/SKILL.md)
- Source refactoring: [../source-refactoring/SKILL.md](../paniolo-source-refactoring/SKILL.md)

## Do Not

- Do not violate repo-wide rules in rules.
- Do not add broad lint/type suppressions without explicit justification.
- Do not expand scope beyond the requested task without calling it out.

## Skill Handoffs

- If reorganizing includes splitting large modules, also load `file-splitting`.
- If symbol naming decisions are part of the reorg, also load `naming-best-practices`.
- **Constants placement (cross-file):** see
  [constants-placement-across-modules](references/constants-placement-across-modules.md).