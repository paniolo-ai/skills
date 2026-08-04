---
source-wiki: sharp-shooter-wiki
source-slug: react-file-and-import-conventions
source-hash: b8044c2a6364717f5eda8726f9a1dc2a123739abbe77c58a00e0f586ee772030
bundled: 2026-08-01
title: File and Import Conventions
type: concept
tags:
- authoring
- react
- client
updated: 2026-06-18
---

# File and Import Conventions

**Strong preference: one component per `.tsx` file.** Keep a single primary component (usually the
default export) per module and colocate its tests in the same directory:

```text
song-form/
  SongForm.tsx
  SongForm.test.tsx
  use-song-form/
    useSongForm.ts
    useSongForm.test.tsx
```

Avoid multiple top-level components in one file—extract siblings (same folder, direct imports) for
tab shells, panel bodies, pickers, and similar. Tiny inline JSX fragments are fine; several named
`function Foo()` / `function Bar()` components in the same module should be split.

**Paired hooks (strong preference):** once a default export picks up non-trivial state, effects, or
domain rules, **extract** them into `use<Component>` in a colocated hook file instead of expanding
the `.tsx` (see [Paired hook for non-display logic](./react-paired-hook-for-non-display-logic.md)). Sibling
shells and bodies each follow that rule when **their** default export carries the logic; thin shells
can stay inline until the logic grows.

**Direct imports — no barrel re-exports.** Import directly from source files; do not create
`index.ts` re-export files:

```tsx
// ❌ Avoid: barrel import hides the real source
import { useSongForm } from "@/song/song-form";

// ✅ Preferred: direct import
import useSongForm from "@/song/song-form/use-song-form/useSongForm";
```

Direct imports make the dependency graph explicit and prevent circular import issues.

## See also

- React (authoring) index