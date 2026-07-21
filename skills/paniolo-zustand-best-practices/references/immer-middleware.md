---
source-wiki: sharp-shooter-wiki
source-slug: immer-middleware
source-hash: 6792952769e8f70fddcd76fc07b7b84d9b86b077eba9b7082ebaa665bb5de300
bundled: 2026-07-20
title: Immer Middleware (Mutable Updates)
type: concept
tags:
- authoring
- zustand
- react
- client
- state
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Immer Middleware (Mutable Updates)

Use Immer when updates involve deeply nested state — it lets you write mutable-looking syntax that
produces immutable updates:

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

## See also

- Zustand (authoring) index