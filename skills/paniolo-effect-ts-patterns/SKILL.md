---
name: paniolo-effect-ts-patterns
description: |
  Effect-TS patterns for scripts (CLI orchestration, subprocesses, typed errors, schema validation). Use when building or migrating scripts, hooks, or validators — not Hono APIs.
license: MIT
metadata:
  version: 0.1.0
tags:
- effect-ts
- typescript
references:
- 'wiki: sharp-shooter-wiki:entry-point-vs-logic-module-pattern → references/entry-point-vs-logic-module-pattern.md'
- 'wiki: sharp-shooter-wiki:typescript-effect-defining-tagged-errors → references/typescript-effect-defining-tagged-errors.md'
- 'wiki: sharp-shooter-wiki:typescript-effect-logic-module-pattern → references/typescript-effect-logic-module-pattern.md'
- 'wiki: sharp-shooter-wiki:typescript-effect-mixing-promises-and-effects → references/typescript-effect-mixing-promises-and-effects.md'
- 'wiki: sharp-shooter-wiki:typescript-effect-refactoring-promise-to-effect → references/typescript-effect-refactoring-promise-to-effect.md'
- 'wiki: sharp-shooter-wiki:typescript-effect-subprocesses → references/typescript-effect-subprocesses.md'
- 'wiki: sharp-shooter-wiki:typescript-effect-throwing-instead-of-effectfail → references/typescript-effect-throwing-instead-of-effectfail.md'
---

**Requires:** file-read, terminal (linting/testing). No network access needed.

# Effect-TS Patterns Skill

## Use When

Use this skill when:

- Adding or migrating scripts to Effect.
- Refactoring `async`/`Promise` script logic to `Effect.gen`, `Effect.tryPromise`, or tagged errors.
- Composing subprocess, file I/O, or script orchestration.

Prefer exposing functions that return an `Effect` instead of raw `Promise` results. Convert
Promise-based boundaries to Effects with `Effect.tryPromise` so downstream code composes and maps
errors using Effect combinators.

Execution workflow:

1. Identify the script boundary (entry `.bun.ts` vs logic module vs shared helper).
2. Keep errors typed when a subsystem has multiple failure modes (`Data.TaggedError`).
3. Use `Effect.tryPromise` with a small subprocess helper for captured stdout/stderr (see
   [typescript-effect-subprocesses](references/typescript-effect-subprocesses.md)).
4. Validate with the project's lint and test commands.

Output requirements:

- Summarize which Effect patterns were applied (gen, tryPromise, try, tagged errors).
- Note any intentional Promise boundaries (e.g. long-lived servers).

## Key Patterns

### 1. Thin entrypoint + Effect.runPromise

See [entry-point-vs-logic-module-pattern](references/entry-point-vs-logic-module-pattern.md).

### 2. Logic modules return Effect

See [typescript-effect-logic-module-pattern](references/typescript-effect-logic-module-pattern.md).

### 3. Subprocesses via a typed helper

See [typescript-effect-subprocesses](references/typescript-effect-subprocesses.md).

### 4. Tagged errors (when warranted)

See [typescript-effect-defining-tagged-errors](references/typescript-effect-defining-tagged-errors.md).

### 5. Promise → Effect refactor steps

See [typescript-effect-refactoring-promise-to-effect](references/typescript-effect-refactoring-promise-to-effect.md).

## Common Pitfalls

Avoid mixing Promise and Effect in the same call chain, throwing inside `Effect.gen`, and bare
`Effect.tryPromise` without `catch`.

See also:
[typescript-effect-mixing-promises-and-effects](references/typescript-effect-mixing-promises-and-effects.md),
[typescript-effect-throwing-instead-of-effectfail](references/typescript-effect-throwing-instead-of-effectfail.md), and
[typescript-effect-refactoring-promise-to-effect](references/typescript-effect-refactoring-promise-to-effect.md).

## Deep Reference

| Doc                                      | Use for                   |
| ---------------------------------------- | ------------------------- |
| [entry-point-vs-logic-module-pattern](references/entry-point-vs-logic-module-pattern.md) | Entry-point vs logic module |
| [typescript-effect-logic-module-pattern](references/typescript-effect-logic-module-pattern.md) | Logic module pattern      |
| [typescript-effect-subprocesses](references/typescript-effect-subprocesses.md) | Subprocess helper         |
| [typescript-effect-defining-tagged-errors](references/typescript-effect-defining-tagged-errors.md) | Tagged errors             |
| [typescript-effect-refactoring-promise-to-effect](references/typescript-effect-refactoring-promise-to-effect.md) | Promise → Effect refactor |
| [typescript-effect-mixing-promises-and-effects](references/typescript-effect-mixing-promises-and-effects.md) | Mixing Promise and Effect |
| [typescript-effect-throwing-instead-of-effectfail](references/typescript-effect-throwing-instead-of-effectfail.md) | Throwing vs Effect.fail   |

## Validation Commands

```bash
pnpm run lint:ts
pnpm test
```