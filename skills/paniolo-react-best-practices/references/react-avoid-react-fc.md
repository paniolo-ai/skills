---
source-slug: react-avoid-react-fc
source-hash: d206ad670dd158eaa0cf1459ef5d5b17102b139c5e8575c2bd79f119d0f31de3
bundled: 2026-10-17
title: Avoid React.FC
type: concept
tags:
- authoring
- react
- client
updated: 2026-06-18
---

# Avoid React.FC

Do not use `React.FC` or `React.FunctionComponent` to type components. Use an explicit prop type and return type annotation instead:

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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
