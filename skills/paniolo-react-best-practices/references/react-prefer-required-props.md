---
source-slug: react-prefer-required-props
source-hash: 24b75bae478373c8cc13f9ae7938e003b6b82a332f317153e809a88d2281e751
bundled: 2026-10-17
title: Prefer Required Props
type: concept
tags:
- authoring
- react
- client
updated: 2026-06-18
---

# Prefer Required Props

Prefer required props over many optional props. Required props make intent explicit, simplify call sites, and avoid surprising runtime behavior:

- **Use explicit presence flags** — pass a boolean like `hasLyrics` rather than making several lyric-related handlers optional.
- **Pass concrete refs/handlers** — supply a noop handler and fallback ref for non-active fields instead of leaving props undefined.
- **Keep optional props rare and documented** — only make a prop optional when there is a clear, documented reason (e.g., legacy compatibility).

```tsx
// ❌ Optional handlers spread across many call sites
type CellProps = { textareaRef?: RefObject<HTMLTextAreaElement>; onSyncSelection?: () => void };

// ✅ Required props with explicit fallbacks at the call site
type CellProps = { textareaRef: RefObject<HTMLTextAreaElement>; onSyncSelection: () => void };

const noop = () => {};
const fallbackRef = useRef<HTMLTextAreaElement | null>(null);
<Cell textareaRef={isLyrics ? lyricsRef : fallbackRef} onSyncSelection={isLyrics ? sync : noop} />;
```

This also avoids `foo?: T` vs `T | undefined` confusion under `exactOptionalPropertyTypes` — see
typescript-exactoptionalpropertytypes-handling.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
