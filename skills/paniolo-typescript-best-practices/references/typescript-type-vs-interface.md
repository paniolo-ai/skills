---
source-slug: typescript-type-vs-interface
source-hash: 155f2178178889c9b2137b33b2f5da6d5d6523a5f03e9fba4c2eaa26205c5a84
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
