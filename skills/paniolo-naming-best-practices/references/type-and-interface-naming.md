---
source-slug: type-and-interface-naming
source-hash: d25178ef3cee14e85bbcfd9d36c5d0f97d8a4cb7d0c1fe29c457c2d12edfbdc4
bundled: 2026-10-17
title: Type and Interface Naming
type: concept
tags:
- authoring
- naming
- typescript
updated: 2026-06-18
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
