---
source-wiki: sharp-shooter-wiki
source-slug: typescript-naming-variable-naming
source-hash: 2ae3f35eae111f81151a4ebaa7c211a703a018a54c08a20d2e1c41f1ff04344c
bundled: 2026-07-24
title: Variable Naming
type: concept
tags:
- authoring
- naming
- typescript
updated: 2026-06-18
---

# Variable Naming

```typescript
// ✅ Booleans — always is*/has*/can* prefix
const isOwner = currentUserId === ownerId;
const hasParticipants = participants.length > 0;
const canManageEvent = isOwner || isEventAdmin;

// ✅ Module-level constants — UPPER_SNAKE_CASE
const MAX_RETRY_COUNT = 3;
const DEFAULT_LANG = "en";

// ✅ Local variables — camelCase
const currentParticipant = participants.find(...);

// ✅ Callbacks stored in variables — handle* prefix
const handleClick = () => { ... };

// ✅ Refs — camelCase with Ref suffix
const currentEventIdRef = useRef<string | undefined>(undefined);
```

## See also

- Naming conventions (authoring) index