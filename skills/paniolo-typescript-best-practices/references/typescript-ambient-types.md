---
source-slug: typescript-ambient-types
source-hash: 0d43b8f05f135818d1b440623c9ba32ba4273ed7be0501aec21d3aa9e039d13a
bundled: 2026-10-17
title: Ambient Types
type: concept
tags:
- authoring
- typescript
updated: 2026-06-18
---

# Ambient Types

`ReactElement` is declared as a global ambient type — never import it. `ReactNode` must be imported:

```typescript
// ❌ Avoid: importing ReactElement (it's ambient)
import type { ReactElement, ReactNode } from "react";

// ✅ Preferred: only import ReactNode
import type { ReactNode } from "react";

function MyComponent({ children }: { children: ReactNode }): ReactElement {
  return <div>{children}</div>;
}
```

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
