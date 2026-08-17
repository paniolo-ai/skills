---
source-wiki: sharp-shooter-wiki
source-slug: testing-zustand-unit-testing
source-hash: dc48e920260e258a3775f14ddc32e245f254f4259747d1d536a86d9070ee24bf
bundled: 2026-08-01
title: Unit Testing
type: concept
tags:
- authoring
- zustand
- react
- client
- state
updated: 2026-06-18
---

# Unit Testing

Reset store state before each test with `setState`:

```typescript
import { act, renderHook } from "@testing-library/react";
import { useCounterStore } from "./counterStore";

describe("useCounterStore", () => {
	beforeEach(() => {
		// Reset to initial state before each test
		useCounterStore.setState({ count: 0 });
	});

	it("increments count", () => {
		// Arrange
		const { result } = renderHook(() => useCounterStore());

		// Act
		act(() => {
			result.current.increment();
		});

		// Assert
		expect(result.current.count).toBe(1);
	});

	it("decrements count", () => {
		// Arrange
		useCounterStore.setState({ count: 2 });
		const { result } = renderHook(() => useCounterStore());

		// Act
		act(() => {
			result.current.decrement();
		});

		// Assert
		expect(result.current.count).toBe(1);
	});
});
```

## See also

- Zustand (authoring) index
