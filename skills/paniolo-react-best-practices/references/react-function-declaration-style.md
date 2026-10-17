---
source-slug: react-function-declaration-style
source-hash: fc8a2405862027d113af712c8037d0a9f8d26a66cf36ac06ec8c23e2088b91c9
bundled: 2026-10-17
title: Function Declaration Style
type: concept
tags:
- authoring
- react
- client
updated: 2026-06-18
---

# Function Declaration Style

Prefer plain function declarations over arrow function assignments for components, event handlers, and local callbacks:

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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
