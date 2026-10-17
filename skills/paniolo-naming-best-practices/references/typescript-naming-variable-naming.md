---
source-slug: typescript-naming-variable-naming
source-hash: f33e5e7184a73239098eacea178382993019aaf4cdaf85f74bfd528d372bb2fe
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
