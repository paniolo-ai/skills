---
source-slug: immer-middleware
source-hash: e0fb618257452b3b4780c794ff4542ca9f0f483a328bf2c2c24d960dea569be6
bundled: 2026-10-16
title: Immer Middleware (Mutable Updates)
type: concept
tags:
- authoring
- zustand
- react
- client
- state
updated: 2026-06-18
---

# Immer Middleware (Mutable Updates)

Use Immer when updates involve deeply nested state — it lets you write mutable-looking syntax that produces immutable updates:

```typescript
import { create } from "zustand";
import { immer } from "zustand/middleware/immer";

type TodoState = {
	todos: Array<{ id: string; title: string; done: boolean }>;
	addTodo: (title: string) => void;
	toggleTodo: (id: string) => void;
	removeTodo: (id: string) => void;
};

export const useTodoStore = create<TodoState>()(
	immer((set) => ({
		todos: [],
		addTodo: (title) =>
			set((state) => {
				state.todos.push({ id: crypto.randomUUID(), title, done: false });
			}),
		toggleTodo: (id) =>
			set((state) => {
				const todo = state.todos.find((t) => t.id === id);
				if (todo) todo.done = !todo.done;
			}),
		removeTodo: (id) =>
			set((state) => {
				state.todos = state.todos.filter((t) => t.id !== id);
			}),
	})),
);
```

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
