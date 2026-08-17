---
source-wiki: sharp-shooter-wiki
source-slug: selector-factory-pattern
source-hash: 89b0a6699bb02c3f7e013636d2fdefff11d8719756042382d2adad8161eab511
bundled: 2026-08-01
title: Selector Factory Pattern
type: concept
tags:
- authoring
- zustand
- react
- client
- state
updated: 2026-06-18
---

# Selector Factory Pattern

Define selectors as named constants outside components to keep them reusable and stable:

```typescript
const todoSelectors = {
	selectAll: (state: TodoState) => state.todos,
	selectDone: (state: TodoState) => state.todos.filter((t) => t.done),
	selectPending: (state: TodoState) => state.todos.filter((t) => !t.done),
	selectCount: (state: TodoState) => state.todos.length,
	selectByStatus: (done: boolean) => (state: TodoState) =>
		state.todos.filter((t) => t.done === done),
};

// Usage
const allTodos = useTodoStore(todoSelectors.selectAll);
const doneTodos = useTodoStore(todoSelectors.selectDone);
const pendingTodos = useTodoStore(todoSelectors.selectByStatus(false));
```

## See also

- Zustand (authoring) index
