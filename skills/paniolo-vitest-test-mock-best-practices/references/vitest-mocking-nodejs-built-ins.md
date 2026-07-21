---
source-wiki: sharp-shooter-wiki
source-slug: vitest-mocking-nodejs-built-ins
source-hash: eadcc5e7d38737fc8f2513d7de66280ee9ab2f98ca83387a0611810a7d1eabf9
bundled: 2026-07-20
title: ❌ Mocking Node.js Built-ins (`node:fs/promises`, `node:path`, etc.)
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

# ❌ Mocking Node.js Built-ins (`node:fs/promises`, `node:path`, etc.)

Do not try to mock `node:fs/promises` or other Node.js core modules with `vi.mock`. Use the real
filesystem with `mkdtemp` / `rm` to set up and tear down actual temporary directories.

```typescript
import { mkdtemp, mkdir, writeFile, rm } from "node:fs/promises";
import os from "node:os";
import path from "node:path";

it("collects only SKILL.md files recursively", async () => {
	const tmp = await mkdtemp(path.join(os.tmpdir(), "my-test-"));
	try {
		await mkdir(path.join(tmp, "sub"), { recursive: true });
		await writeFile(path.join(tmp, "SKILL.md"), "");
		await writeFile(path.join(tmp, "sub", "SKILL.md"), "");
		const result = await myFn(tmp);
		expect(result.toSorted()).toStrictEqual(
			[path.join(tmp, "SKILL.md"), path.join(tmp, "sub", "SKILL.md")].toSorted(),
		);
	} finally {
		await rm(tmp, { recursive: true, force: true });
	}
});
```

## See also

- Vitest mocking (authoring) index