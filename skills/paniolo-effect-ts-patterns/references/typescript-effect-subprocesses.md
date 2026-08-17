---
source-wiki: sharp-shooter-wiki
source-slug: typescript-effect-subprocesses
source-hash: d51d1e3f5ce6ce7aceb661ad849c98f2fe5cdda393af88dc9b41be23dabae734
bundled: 2026-08-01
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
