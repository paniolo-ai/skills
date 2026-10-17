---
source-slug: typescript-naming-use-vs-compute
source-hash: e00a0f64a16b6d906e782f2f0ed164672bda25f5e142758e2549fd88ee508630
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
