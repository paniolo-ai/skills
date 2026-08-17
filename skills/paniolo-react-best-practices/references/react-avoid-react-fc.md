---
source-wiki: sharp-shooter-wiki
source-slug: react-avoid-react-fc
source-hash: f84e8ae0930c85a37edd32f3524fe8a9c61aa9f7f631a4a0181ea1b0efd8f0f4
bundled: 2026-08-01
title: Avoid React.FC
type: concept
tags:
- authoring
- react
- client
updated: 2026-06-18
---

# Avoid React.FC

Do not use `React.FC` or `React.FunctionComponent` to type components. Use an explicit prop type and
return type annotation instead:

```tsx
// ❌ Avoid
const MyComponent: React.FC<MyComponentProps> = ({ value }) => <div>{value}</div>;

// ✅ Preferred
export function MyComponent({ value }: MyComponentProps): ReactElement {
	return <div>{value}</div>;
}
```

`React.FC` implicitly widens the return type and historically added implicit `children` to all
components. Explicit types make the contract clear and consistent with the rest of the codebase.

## See also

- React (authoring) index
