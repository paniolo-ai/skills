---
source-wiki: sharp-shooter-wiki
source-slug: typescript-ambient-types
source-hash: f85771e9d32baaacbb3860d65632d114f2d1f1ae2b7a87131f02a90b0b8999e7
bundled: 2026-08-01
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

## See also

- TypeScript (authoring) index