---
source-wiki: sharp-shooter-wiki
source-slug: entry-point-vs-logic-module-pattern
source-hash: 3e31c68ed9aec6cf817cc17ab1622a5d5d070c89fc182b940afaa9e5a5f8ecfd
bundled: 2026-07-20
title: Entry-point vs logic-module pattern
type: concept
tags:
- authoring
- scripts
- effect-ts
updated: 2026-07-20
sources: []
---

# Entry-point vs logic-module pattern

Entry points are thin shells: resolve the repo root, call a pure exported function, and handle
side-effects only (`process.exit`, `stdout`/`stderr`). All non-trivial logic lives in a pure
module that is unit-testable without spawning a separate process.

```ts
// cli.bun.ts  ← thin shell
import path from "node:path";
import { checkFiles } from "./checkFiles.ts";

async function main(): Promise<void> {
	const repoRoot = path.resolve(import.meta.dir, "..");
	const result = await checkFiles(repoRoot);
	if (result.hasError) {
		for (const message of result.errors) {
			process.stderr.write(`✗ ${message}\n`);
		}
		process.exit(1);
	}
	process.stdout.write(`✓ ${result.checkedCount} file(s) pass.\n`);
}
await main();
```

The pure module returns a typed result object instead of exiting:

```ts
// checkFiles.ts  ← pure, testable
export type CheckFilesResult = {
	hasError: boolean;
	checkedCount: number;
	errors: string[];
};
export async function checkFiles(repoRoot: string): Promise<CheckFilesResult> {
	/* ... */
}
```

## Effect entrypoints

For Effect-based scripts, the entry-point shell calls `Effect.runPromise` instead of `async main()`:

```typescript
#!/usr/bin/env bun
import { resolve } from "node:path";

import { Effect } from "effect";

import runSkills from "./runSkills.ts";

const ARGS_OFFSET = 2;
const repoRoot = resolve(import.meta.dir, "../..");
const args = process.argv.slice(ARGS_OFFSET);

const exitCode = await Effect.runPromise(runSkills(args, { repoRoot }));
process.exit(exitCode);
```

Do not put orchestration logic in the entry-point file — keep it in a default-export function in a
sibling module. See [Logic module pattern](./typescript-effect-logic-module-pattern.md).

## See also

- Logic module pattern
- Refactoring Promise to Effect