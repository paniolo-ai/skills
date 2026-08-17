---
source-wiki: sharp-shooter-wiki
source-slug: typescript-naming-use-vs-compute
source-hash: 8d7251c54f0e4f9b55898909cbd869720ca62c5074ea73f95ee5c9bb0b0f75d4
bundled: 2026-08-01
title: use\* vs compute\* — The Most Common Mistake
type: concept
tags:
- authoring
- naming
- typescript
updated: 2026-06-18
---

# use\* vs compute\* — The Most Common Mistake

```typescript
// ❌ BAD: named like a hook but has no React hooks inside
export default function useEventPermissions({ currentUserId, ownerId, participants }) {
  const isOwner = currentUserId === ownerId;
  // ... pure computation, no useState/useEffect/etc.
  return { isOwner, canManageEvent };
}

// ✅ GOOD: rename to compute* to signal it is a pure function
export default function computeEventPermissions({ currentUserId, ownerId, participants }) {
  const isOwner = currentUserId === ownerId;
  return { isOwner, canManageEvent };
}

// ✅ GOOD: use* is correct when React hooks are called
export default function useEventManageView() {
  const [actionState, setActionState] = useState<ActionState>(...);
  useEffect(() => { ... }, []);
}
```

## See also

- Naming conventions (authoring) index
