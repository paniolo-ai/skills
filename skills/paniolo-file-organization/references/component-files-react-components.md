---
source-slug: component-files-react-components
source-hash: 80e9117fa7bafe116e2b2a8130a02cf4f12a809c5e3c8ee72e222d8ff1a696f2
bundled: 2026-10-17
title: Component Files (React Components)
type: concept
tags:
- authoring
- file-organization
updated: 2026-06-18
---

# Component Files (React Components)

> **Hub:** [File organization (authoring)](./file-organization.md)

**PascalCase** — one component per file is a **strong preference**: each module should expose one primary component (typically the default export). Avoid stacking several top-level `function` components in a single `.tsx`; split into sibling files in the same feature folder instead.

```text
src/components/
├── SongCard.tsx          # Main export: export function SongCard
├── SongCard.test.tsx     # Colocated test
├── UserMenu.tsx
├── UserMenu.test.tsx
└── PlaybackControls.tsx
```

**Pattern:** Match filename exactly to main export name.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
