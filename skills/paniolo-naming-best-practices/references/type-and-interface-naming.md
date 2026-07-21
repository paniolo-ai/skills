---
source-wiki: sharp-shooter-wiki
source-slug: type-and-interface-naming
source-hash: 8707ff609ad5c424a2bc86df66a4418e0026074c89648a3c477369dccf165332
bundled: 2026-07-20
title: Type and Interface Naming
type: concept
tags:
- authoring
- naming
- typescript
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Type and Interface Naming

```typescript
// Props for React components — suffix *Props
type EventManageViewProps = { ... };

// Return shapes — suffix *Return or *Result
type ComputeEventPermissionsReturn = { ... };
type UseEventManageStateResult = { ... };

// Error types — suffix *Error
class DatabaseError { ... }
type EventError = "validation" | "not_found" | "unauthorized";

// Slice state — suffix *Slice or *State
type EventSlice = EventState & { ... };

// Generic type parameters — single uppercase or short descriptive PascalCase
type Selector<TState, TValue> = (state: TState) => TValue;
```

## See also

- Naming conventions (authoring) index