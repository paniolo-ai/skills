---
source-wiki: sharp-shooter-wiki
source-slug: typescript-quick-lookup
source-hash: adf9ef1394cb5f20dbedf727527a97d3992b596ce5b360615539fcbeb5f6cccb
bundled: 2026-07-20
title: Quick Lookup — Common Errors
type: concept
tags:
- authoring
- typescript
- lint
- eslint
updated: 2026-06-18
sources:
- raw/harness-eng/doc-fragment-extraction/doc-content-organization.md
---

# Quick Lookup — Common Errors

| Error                                | Fix                                                      |
| ------------------------------------ | -------------------------------------------------------- |
| `typescript/no-explicit-any`         | `unknown` + narrow.                                      |
| `no-unsafe-type-assertion`           | Narrow first; `as unknown as T` only with comment.       |
| `consistent-type-imports`            | `import type { … }`.                                     |
| `oxc/no-barrel-file`                 | Import from source module.                               |
| `id-length`                          | Longer name (2+ chars).                                  |
| `no-magic-numbers`                   | Named constant.                                          |
| `no-negated-condition`               | Positive branch first.                                   |
| `unicorn/prefer-number-properties`   | `Number.parseInt()` / `Number.isNaN()`.                  |
| `unicorn/no-array-sort`              | `.toSorted()`.                                           |
| `unicorn/catch-error-name`           | Rename catch param to `error`.                           |
| `curly`                              | Add braces.                                              |
| `func-style`                         | `function foo()` not `const foo = () =>`.                |
| `require-own-file-for-long-function` | Split long module-scope function to its own file.        |
| `require-aaa-pattern`                | Add `// Arrange`, `// Act`, `// Assert` markers.         |
| `no-assert-mocked-return`            | `expect(mock).toHaveBeenCalled()`, not `expect(mock())`. |

## See also

- TypeScript lint (authoring) index