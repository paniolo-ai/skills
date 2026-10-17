---
source-slug: selector-factory-pattern
source-hash: a2a4cb9d095b2d3c5b48a17dce26fb4ed6ae4294a89b654923bcf8e8a65a3296
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
