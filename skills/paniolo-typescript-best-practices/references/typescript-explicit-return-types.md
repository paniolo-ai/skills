---
source-wiki: sharp-shooter-wiki
source-slug: typescript-explicit-return-types
source-hash: 1b3db661d4fb93cfd5d718a787de4be18bfa7462e38efc1906547e073d3990d2
bundled: 2026-07-20
title: Explicit Return Types
type: concept
tags:
- authoring
- typescript
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Explicit Return Types

Always specify explicit return types on functions and React components:

```typescript
// ❌ Avoid: inferred return type
const calculateTotal = (items: Item[]) => {
	return items.reduce((sum, item) => sum + item.price, 0);
};

// ✅ Preferred: explicit return type
const calculateTotal = (items: Item[]): number => {
	return items.reduce((sum, item) => sum + item.price, 0);
};
```

For React components, use the ambient `ReactElement` (no import needed):

```typescript
// ✅ Preferred: ReactElement is ambient — no import needed
function MyComponent({ title }: Props): ReactElement {
  return <div>{title}</div>;
}

// ❌ Avoid: importing ReactElement
import type { ReactElement } from "react"; // unnecessary

// ❌ Avoid: JSX.Element (less precise)
function MyComponent(): JSX.Element { ... }
```

## See also

- TypeScript (authoring) index