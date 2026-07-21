---
source-wiki: sharp-shooter-wiki
source-slug: typescript-exactoptionalpropertytypes-handling
source-hash: 2934dd0f29d75aa9532d36c0e8066f03a3791536e74a8319790d2b7402b08ae8
bundled: 2026-07-20
---
---
title: "exactOptionalPropertyTypes Handling"
type: concept
tags: [authoring, typescript]
updated: 2026-06-18
sources:
  - raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
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
[docs/client/react-best-practices.md](@songshare-effect/docs/client/react-best-practices.md).

## See also

- TypeScript (authoring) index
