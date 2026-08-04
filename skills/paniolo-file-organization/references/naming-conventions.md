---
source-wiki: sharp-shooter-wiki
source-slug: naming-conventions
source-hash: 8dd1fb6df7567ce659af138460f7690299017a60e4e43739870162805b9f2f38
bundled: 2026-08-01
title: Naming Conventions
type: concept
tags:
- authoring
- file-organization
updated: 2026-06-18
---

# Naming Conventions

### Component Files (React Components)

**PascalCase** — one component per file is a **strong preference**: each module should expose one
primary component (typically the default export). Avoid stacking several top-level `function`
components in a single `.tsx`; split into sibling files in the same feature folder instead.

```text
src/components/
├── ItemCard.tsx          # Main export: export function ItemCard
├── ItemCard.test.tsx     # Colocated test
├── UserMenu.tsx
├── UserMenu.test.tsx
└── PlaybackControls.tsx
```

**Pattern:** Match filename exactly to main export name.

### Constants placement across modules

- **1 file:** module-level `const` in that file (no `constants.ts`)
- **2 files:** ask owner — colocate if tightly coupled, otherwise create shared module
- **3+ files:** create `constants.ts` or `feature-constants.ts` (name descriptively)

Rationale: keeps API surface minimal, reduces unnecessary indirection.

### Utility & Function Files

**camelCase** — use for any module that **default-exports a single function** (application code
under `react/`, `api/`, `shared/`, etc.). **Do not** use kebab-case basenames for those files;
kebab-case is reserved for **multi-symbol** `.ts` modules.

```text
src/utils/
├── formatDate.ts         # export default function formatDate(...)
├── formatDate.test.ts
├── parseDuration.ts
├── calculatePlaybackTime.ts
└── validateItemInput.ts
```

**Pattern:** basename matches the default export (`formatDate.ts` → `formatDate`). **`verb` +
`noun`** where applicable (`format`, `calculate`, `validate`, `parse`).

#### Avoid multi-function `*Helpers.ts` modules

Do **not** create application modules such as `activePublicItemSubscriptionHelpers.ts` that export
several named functions. That pattern is legacy; new work should use **one file per function**:

```text
src/item/active-items/
├── mergeActivePublicItemIds.ts
├── mergeActivePublicItemIds.test.ts
├── ensureActivePublicItemSubscription.ts
└── ensureActivePublicItemSubscription.test.ts
```

If you touch an existing multi-export helper file, prefer splitting it (see
[`file-splitting`](../../paniolo-file-splitting/SKILL.md)) over adding another export.

### Playwright E2E util modules (`*.e2e-util.ts`)

**camelCase** basename when the file **default-exports one function**, matching that symbol:

```text
e2e/utils/live/
├── createPublicLiveCommunityChain.e2e-util.ts
├── runEffect.e2e-util.ts
└── slideRootTextMatchesUserNotEvent.e2e-util.ts
```

Do not use kebab-case basenames for single-function default exports. Multi-export or type-focused
helpers follow the same multi-symbol / `.type` patterns as application code (see
`.agents/rules.md`).

### Type & Interface Files

**`PascalCase.type.ts`** when the module exports **exactly one** type (and no runtime exports); the
basename matches the type name (for example `ShareCreateRequest.type.ts`). **`kebab-case.type.ts`**
when **all** exports are types but there are **multiple** symbols (see `.agents/rules.md`).

**camelCase.ts** for types, or **types.ts** in feature folders (multi-type or informal groupings):

```text
src/types/
├── item.ts               # export type Item, type ItemLibrary, ...
├── api.ts                # export type ApiResponse, type ErrorResponse, ...
└── ui.ts                 # export type ButtonProps, type ModalProps, ...

react/src/auth/
├── auth-types.ts         # export type AuthState, type User, ...
└── auth-slice.ts
```

### Hook Files

**useHookName.ts** - Always start with `use`:

```text
src/hooks/
├── useItemLibrary.ts     # export function useItemLibrary() ...
├── useItemLibrary.test.ts
├── usePlaybackState.ts
└── useAuth.ts
```

#### Paired hooks (`use<ComponentName>`)

When a hook exists solely to power **one** component — holding that component's state, derived
values, and handlers so the `.tsx` stays presentational — name it `use<ComponentName>` (matching the
component exactly) and **colocate** it as a sibling file in the same feature folder. Do not place it
in a generic/shared folder. The colocated `.test.tsx` mirrors the hook name.

```text
src/tools/tabs/content/item-details/
├── BroadcastItemButton.tsx         # export default function BroadcastItemButton
├── BroadcastItemButton.test.tsx
├── useBroadcastItemButton.ts       # paired hook for BroadcastItemButton
├── useBroadcastItemButton.test.tsx
├── ItemDetailToolsPanelBody.tsx
└── useItemDetailToolsPanelBody.ts  # paired hook for ItemDetailToolsPanelBody
```

A hook reused by **multiple** components is not a paired hook — name it for its behavior
(`useBroadcastTabLibraryFetch`) and place it in the relevant feature folder.

### Feature/Domain Directories

**kebab-case** - Group related files:

```text
src/
├── auth/                 # Authentication feature
│   ├── auth-slice.ts
│   ├── auth-slice.test.ts
│   ├── useAuth.ts
│   ├── useAuth.test.ts
│   └── types.ts
├── item-library/         # Item library feature
│   ├── ItemLibrary.tsx
│   ├── ItemLibrary.test.tsx
│   ├── useItemLibrary.ts
│   └── constants.ts
└── playback/             # Playback feature
    ├── PlaybackControls.tsx
    └── usePlayback.ts
```

### Documentation Files

**kebab-case** - All documentation:

```text
docs/
├── authentication-system.md
├── effect-implementation.md
├── commit-message-instructions.md
├── component-patterns.md
└── file-organization.md
```

### Configuration Files

**Preserve original, use ESM** (not CommonJS):

```text
vite.config.ts           # ESM: export default { ... }
tailwind.config.js       # ESM: export default { ... }
tsconfig.json            # JSON (not .js)
commitlint.config.js     # ESM: export default { ... }
```

## See also

- File organization (authoring) index