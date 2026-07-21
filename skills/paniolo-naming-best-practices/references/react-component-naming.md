---
source-wiki: sharp-shooter-wiki
source-slug: react-component-naming
source-hash: b1fe423b2433ddd6c57d9af1c770430572a67beffd547938d6128125058d83f3
bundled: 2026-07-20
title: React Component Naming
type: concept
tags:
- authoring
- naming
- typescript
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
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

## See also

- Naming conventions (authoring) index