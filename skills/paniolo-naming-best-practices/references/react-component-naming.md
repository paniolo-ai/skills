---
source-slug: react-component-naming
source-hash: 202df3fddb2a4bf549a6e83ad82aa6d901bde019b488e95fd6f9e35bd0afea6f
bundled: 2026-10-17
title: React Component Naming
type: concept
tags:
- authoring
- naming
- typescript
updated: 2026-06-18
---

# React Component Naming

```typescript
// ✅ Components — PascalCase, matches filename
export default function EventManageView(): ReactElement { ... }
// file: EventManageView.tsx

// ✅ Context providers — suffix *Provider
export default function AuthProvider({ children }: { children: ReactNode }) { ... }
// file: AuthProvider.tsx

// ✅ Compound/sub-components — PascalCase, descriptive
function ParticipantRow({ participant }: ParticipantRowProps) { ... }
```

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
