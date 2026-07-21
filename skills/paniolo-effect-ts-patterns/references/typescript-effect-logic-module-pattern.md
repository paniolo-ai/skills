---
source-wiki: sharp-shooter-wiki
source-slug: typescript-effect-logic-module-pattern
source-hash: e11437e2e8dc788e112365713e58f48f7560fb18770b9c67caf37b9587957c14
bundled: 2026-07-20
title: Logic module pattern
type: concept
tags:
- authoring
- effect-ts
- scripts
updated: 2026-07-20
sources: []
---

# Logic module pattern

Use `Effect.gen` for sequential steps and `Effect.tryPromise` / `Effect.try` at boundaries:

```typescript
import { Effect } from "effect";

import { runProcess } from "./runProcess.ts";

export type CheckResult = {
	readonly hasError: boolean;
	readonly checkedCount: number;
	readonly errors: readonly string[];
};

/**
 * Validate files under the repo.
 *
 * @param repoRoot - Repository root.
 * @returns Effect yielding a structured check result.
 */
export default function checkFiles(repoRoot: string): Effect.Effect<CheckResult, Error> {
	return Effect.gen(function* checkFilesGen() {
		const files = yield* collectFiles(repoRoot);
		// …
		return { hasError: false, checkedCount: files.length, errors: [] };
	});
}
```

Convert existing `async function …(): Promise<T>` using the steps in
[Refactoring Promise-Returning Functions to Effect](./typescript-effect-refactoring-promise-to-effect.md).

## See also

- Subprocesses
- Refactoring Promise to Effect