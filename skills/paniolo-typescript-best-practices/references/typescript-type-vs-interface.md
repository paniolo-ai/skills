---
source-wiki: sharp-shooter-wiki
source-slug: typescript-type-vs-interface
source-hash: 669379749ab79c9b44ace7cfae72b0fe20c5e4c3399fbe3f6d7cdfb213fd13ea
bundled: 2026-07-24
title: Type vs Interface
type: concept
tags:
- authoring
- typescript
updated: 2026-06-18
---

# Type vs Interface

Prefer `type` over `interface` for consistency:

```typescript
// ✅ Preferred: type alias
type UserProps = {
	name: string;
	email: string;
};

// ❌ Avoid: interface for plain object shapes
interface UserProps {
	name: string;
	email: string;
}
```

Use `interface` only when extending built-in or external types:

```typescript
// ✅ Valid: extending a browser API type
interface AudioTrack extends MediaStreamTrack {
	gainNode: GainNode;
}
```

## See also

- TypeScript (authoring) index