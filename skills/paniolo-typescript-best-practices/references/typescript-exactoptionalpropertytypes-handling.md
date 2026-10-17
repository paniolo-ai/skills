---
source-slug: typescript-exactoptionalpropertytypes-handling
source-hash: 3d2a98c97ba083778559c4894d5f04a194015ad30a944a586eba328d17cffa15
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
