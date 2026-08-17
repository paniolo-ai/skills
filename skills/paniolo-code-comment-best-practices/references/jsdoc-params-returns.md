---
source-wiki: sharp-shooter-wiki
source-slug: jsdoc-params-returns
source-hash: e51b98e40d05fefc61dea09d353e99b00d5fe3f062ca8df375d21f76d35d66a8
bundled: 2026-08-01
title: JSDoc — params and returns
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
---

# JSDoc — params and returns

**Always include `@returns`.** For `void` functions, write `@returns void` so the intent is
explicit.

**Single-object params (props) — document fields directly.** When a function accepts a single object
(often `props`), do not create a standalone `@param props` entry. List each field as its own
`@param`. This matches destructuring call-site usage and avoids misleading noise.

```ts
// ❌
/**
 * Small interactive demo component.
 *
 * @param props - Component props
 * @param props.title - Title to display
 * @param props.colSpan - Number of columns to span
 * @returns React element
 */

// ✅
/**
 * Small interactive demo component.
 *
 * @param title - Title to display
 * @param colSpan - Number of columns to span
 * @returns React element
 */
```

**Explicit rule:** Never document the wrapper object itself (for example `@param options`) when the
function takes a single object — document its fields directly.

Bad and good examples for object-style options:

```ts
// ❌ — do not do this
/**
 * Bootstraps the local QMD index for a command.
 *
 * @param options - Configuration for bootstrapping the local index.
 * @param options.command - The command name used to determine whether bootstrapping is needed.
 * @param options.indexPath - Path to the local index file to create or verify.
 * @param options.globalIndexPath - Path to a shared/global index file that can be copied into place.
 * @param options.qmdBin - Path to the `qmd` binary used to run update commands.
 * @returns void
 */

// ✅ — preferred style
/**
 * Bootstraps the local QMD index for a command.
 *
 * @param command - The command name used to determine whether bootstrapping is needed.
 * @param indexPath - Path to the local index file to create or verify.
 * @param globalIndexPath - Path to a shared/global index file that can be copied into place.
 * @param qmdBin - Path to the `qmd` binary used to run update commands.
 * @returns void
 */
```

**Object return values — document properties directly.** Do not write a top-level return object
description. List returned properties explicitly:

```ts
// ❌
 * @returns UseSongViewResult - object with the fields described below

// ✅
 * @returns isNotFound - true when the slug did not resolve to a song
 * @returns songData - the raw store payload or `undefined` when not found
 * @returns songPublic - the validated payload, or `undefined` if validation failed
```

**Full example:**

```ts
/**
 * Renders the inner TD content for the full-width delete confirmation UI.
 * Maintainers: This avoids alignment shifts seen in separate-row implementations.
 *
 * @param colSpan - number of columns to span across the grid
 * @returns React element (TD)
 */
```

## See also

- Code comments (authoring) index
