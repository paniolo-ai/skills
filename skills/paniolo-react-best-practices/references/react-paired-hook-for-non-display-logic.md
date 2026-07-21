---
source-wiki: sharp-shooter-wiki
source-slug: react-paired-hook-for-non-display-logic
source-hash: c78878bf923937eb6d0846512ee9a71146fa8d6f08c74ae9f0553e1a80634378
bundled: 2026-07-20
title: Paired hook for non-display logic
type: concept
tags:
- authoring
- react
- client
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Paired hook for non-display logic

**Strong preference:** default-export `.tsx` modules that own **non-trivial** non-display logic
(state beyond a single trivial flag, coordinated `useEffect`s, refs that drive behavior across
renders, router or URL sync, store subscriptions, inline-edit controllers, or domain rules) should
**extract** that work into a **colocated hook** named **`use` + the exact component identifier**
(see the table below). Do not keep growing those concerns inline in the component file when a
paired hook would isolate them.

Move behavior and derivation into the paired hook:

- Other hooks (`useAppStore`, `useLocation`, `useNavigate`, data hooks, feature hooks, etc.).
- Local state, refs, effects, subscriptions, async operations, and mutation orchestration.
- Event handlers and named functions that coordinate behavior (`handleSave`, `handleDelete`, etc.).
- Derived constants that are not styling: ownership booleans, selected records, hrefs, labels from
  domain data, badge variants, permission flags, and normalized values.
- Domain calculations and data shaping needed before render.

Keep display-only concerns in the component:

- JSX structure, conditional rendering shape, icons, and visual composition.
- `className`, Tailwind strings, design-token choices, and style/layout constants.
- Copy rendering through `t(...)` when the component owns the visible text.

| Component (default export) | Hook module                 | Hook export              |
| -------------------------- | --------------------------- | ------------------------ |
| `SignedInToolsPanel`       | `useSignedInToolsPanel.ts`  | `useSignedInToolsPanel`  |
| `MySettingsCard`           | `useMySettingsCard.ts`      | `useMySettingsCard`      |
| `ToolsTabsSlidePanel`      | `useToolsTabsSlidePanel.ts` | `useToolsTabsSlidePanel` |

The hook basename is **`use` + the same PascalCase identifier as the component** (camelCase file
name per [file conventions](./react-file-and-import-conventions.md)). The `.tsx` file should stay focused on
JSX: call the hook once, destructure what you need, render.

**One paired hook per component.** Do not add a second hook call in the `.tsx` for shell chrome,
banner visibility, badge placement flags, or other derived layout rules tied to that screen. Fold
those derivations into the existing `use<Component>` return (for example `showBannerRow`,
`hasBannerActions`, and live-badge inputs on `useToolsTabsSlidePanel`) instead of introducing a
sibling hook such as `useToolsTabsSlidePanelBanner` that the component must call separately.

When a screen is split across sibling files (for example a tools-tab `FooContent.tsx` shell and a
`FooBody.tsx` view—[one component per file](./react-file-and-import-conventions.md)), each **default export**
that owns substantial logic **should** pair with its own hook. Follow the same shell-vs-body split
as Split hooks over prop spreading: keep props between them
minimal and intentional. Thin shells that only resolve route or tab inputs and render a child may
stay inline without a dedicated hook until that logic becomes non-trivial.

**Do not accept, return, or prop-drill `t`.** Components that render copy call `useLocale()` or
`useTranslation()` themselves and keep `t(...)` beside the JSX. Paired hooks may also call locale
hooks for non-display needs such as `lang`-aware href building, routing, or hook-owned derived
strings, but do not move `t` out of a rendering component just to centralize it. **Never** pass `t`
as a prop (for example `t={t}`) or a hook argument (for example `useThing({ t })`).

Export a **`Use<ComponentName>Return`** (or similar) readonly type for the hook result when the
return shape is large enough to help call sites and tests.

Pure helpers that do not need React APIs belong in separate modules; the paired hook is for
**stateful / effectful** behavior tied to that screen or widget.

## See also

- React (authoring) index