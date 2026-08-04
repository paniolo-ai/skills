---
source-wiki: sharp-shooter-wiki
source-slug: typescript-exactoptionalpropertytypes-handling
source-hash: 6dbe8957673d78e308d8a8d4a991a6aa3187451114dd50d1a7aa8b4534bdb146
bundled: 2026-08-01
title: exactOptionalPropertyTypes Handling
type: concept
tags:
- authoring
- typescript
updated: 2026-06-18
---

# exactOptionalPropertyTypes Handling

**Challenge**: Optional properties can't receive `T | undefined`, only `T` or omission.

**Solution**: Conditional spread pattern:

```typescript
// ❌ FAILS: passes `string | undefined` where `string?` expected
function MyComponent({ optionalProp }: { optionalProp?: string }) {
  return <ChildComponent optionalProp={optionalProp} />; // Type error!
}

// ✅ GOOD: conditional spread (no-negated-condition safe form)
function MyComponent({ optionalProp }: { optionalProp?: string }) {
  return (
    <ChildComponent
      {...(optionalProp === undefined ? {} : { optionalProp })}
    />
  );
}
```

For UI-specific best-practices (component props, prop patterns, and React conventions), see
your repo's React best-practices guidance.

## See also

- TypeScript (authoring) index