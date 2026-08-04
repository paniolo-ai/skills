---
source-wiki: sharp-shooter-wiki
source-slug: react-function-declaration-style
source-hash: b5e2c4a1856b18318870a0e06ee581568e78c7b1913ba222d9c144e364655d54
bundled: 2026-08-01
title: Function Declaration Style
type: concept
tags:
- authoring
- react
- client
updated: 2026-06-18
---

# Function Declaration Style

Prefer plain function declarations over arrow function assignments for components, event handlers,
and local callbacks:

```tsx
// ❌ Avoid: arrow function assignment
const handleClick = () => {
	doSomething();
};

export const MyComponent = ({ value }: MyComponentProps): ReactElement => <div>{value}</div>;

// ✅ Preferred: plain function declaration
function handleClick(): void {
	doSomething();
}

export function MyComponent({ value }: MyComponentProps): ReactElement {
	return <div>{value}</div>;
}
```

Function declarations are hoisted, show up cleanly in stack traces, and are consistent with the
project's hook and utility file style.

## See also

- React (authoring) index