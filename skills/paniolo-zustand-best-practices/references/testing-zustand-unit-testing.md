---
source-slug: testing-zustand-unit-testing
source-hash: 3e30f6ccf06ab3200ac05673ebd4a40f0744054143d145ae8981271fc98e16e0
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
