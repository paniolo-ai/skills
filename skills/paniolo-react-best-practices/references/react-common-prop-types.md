---
source-slug: react-common-prop-types
source-hash: 0c8bb3fc79ae9e742904d727e171ab94914d64184694913d62fb292164500261
bundled: 2026-10-17
title: Common Prop Types
type: concept
tags:
- authoring
- react
- client
updated: 2026-06-18
---

# Common Prop Types

**`ReactNode` for children and renderable content:**

Use `ReactNode` for props that accept any renderable React content:

```tsx
type CardProps = {
	children: ReactNode; // any renderable content
	header?: ReactNode; // optional renderable content
	footer: ReactNode | undefined; // required but can be explicitly undefined
};
```

**`ReactElement` for specific component instances:**

Use `ReactElement` when you need a JSX element, not text or primitives:

```tsx
type ModalProps = {
	trigger: ReactElement; // must be a React element
	content: ReactNode; // can be any renderable content
};
```

**When to use each:**

- `ReactNode` — `children` and any content that can include text, numbers, or elements
- `ReactElement` — when you specifically need a JSX element (component instance)
- `ReactElement` — for component return types
- `string` — when only text is valid (e.g., `label`, `title`)

**Import note:** `ReactElement` is ambient in projects configured for it (no import needed). `ReactNode` must be
imported:

```tsx
import type { ReactNode } from "react";
```

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
