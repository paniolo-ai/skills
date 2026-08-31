---
source-slug: typescript-effect-subprocesses
source-hash: 6c44ad7728cdc460efd1bb8ef92d23e6b7bad70c0e6f6a7ac39be1d129d52a0b
bundled: 2026-08-30
title: Subprocesses
type: concept
tags:
- authoring
- effect-ts
- scripts
updated: 2026-07-20
---

# Subprocesses

Use `Effect.tryPromise` with a small subprocess helper. The helper should capture stdout/stderr and
return a typed result:

```typescript
import { Effect } from "effect";
import { spawn } from "node:child_process";

export const runProcess = (
	command: string[],
	cwd?: string
): Effect.Effect<{ exitCode: number; output: string }, Error> =>
	Effect.tryPromise(
		() =>
			new Promise((resolve, reject) => {
				const [cmd, ...args] = command;
				const child = spawn(cmd, args, { cwd });
				let output = "";
				child.stdout.on("data", (data) => {
					output += data.toString();
				});
				child.stderr.on("data", (data) => {
					output += data.toString();
				});
				child.on("error", reject);
				child.on("close", (exitCode) => {
					resolve({ exitCode: exitCode ?? 0, output });
				});
			})
	);
```

For subprocesses that must run in a sibling directory, pass the target `cwd` to the helper:

```typescript
const { exitCode, output } = yield* runProcess(["pnpm", "run", "lint:md"], repoRoot);
```

For inherited stdio (interactive CLI), keep `spawnSync` or a dedicated helper; do not duplicate
spawn logic across the codebase.

## See also

- Logic module pattern
- Refactoring Promise to Effect
