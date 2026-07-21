---
source-wiki: sharp-shooter-wiki
source-slug: vitest-react-dom-mocking-flushsync
source-hash: 5d8076356a0d4533913e1453322ccdbbd67dd0c0e9438a44e2fb95c54e396ed6
bundled: 2026-07-20
title: '`react-dom` mocking (`flushSync`)'
type: concept
tags:
- authoring
- vitest
- testing
- mocking
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# `react-dom` mocking (`flushSync`)

If the SUT uses `flushSync` from `react-dom` to force synchronous state updates, you must mock it
in tests to prevent "cannot find react-dom" or runtime errors in non-browser environments.

To avoid the `jest.mock() factories should not be used without an explicit type parameter` lint
error, follow the non-factory `vi.mock()` pattern and implement the mock behavior in each test.

```tsx
import { flushSync } from "react-dom";
import { describe, expect, it, vi } from "vitest";

// ✅ PREFERRED: Non-factory mock
vi.mock("react-dom");

describe("myComponent", () => {
	it("forces synchronous updates", () => {
		// Arrange
		vi.mocked(flushSync).mockImplementation(<TReturn>(cb: () => TReturn): TReturn => cb());
		// ...
		// Assert
		expect(flushSync).toHaveBeenCalledWith(expect.any(Function));
	});
});
```

## See also

- Vitest mocking (authoring) index