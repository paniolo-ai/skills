---
source-wiki: sharp-shooter-wiki
source-slug: component-files-react-components
source-hash: 2f7c2c37f1a39e3701ae2b0571f179a558d69b1e1f5661997b6e6477bfb004dd
bundled: 2026-08-01
title: Component Files (React Components)
type: concept
tags:
- authoring
- file-organization
updated: 2026-06-18
---

# Component Files (React Components)

> **Hub:** File organization (authoring) index

**PascalCase** — one component per file is a **strong preference**: each module should expose one
primary component (typically the default export). Avoid stacking several top-level `function`
components in a single `.tsx`; split into sibling files in the same feature folder instead.

```text
src/components/
├── SongCard.tsx          # Main export: export function SongCard
├── SongCard.test.tsx     # Colocated test
├── UserMenu.tsx
├── UserMenu.test.tsx
└── PlaybackControls.tsx
```

**Pattern:** Match filename exactly to main export name.

## See also

- File organization (authoring) index

## See also

- File organization (authoring) index