---
source-slug: typescript-explicit-return-types
source-hash: ff5db1e6f1f72868693527636c15585e2301e5c07faf0b5243492538d8e14add
bundled: 2026-10-17
title: Explicit Return Types
type: concept
tags:
- authoring
- typescript
updated: 2026-06-18
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
