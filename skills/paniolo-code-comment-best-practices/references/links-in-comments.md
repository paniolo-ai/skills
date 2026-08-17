---
source-wiki: sharp-shooter-wiki
source-slug: links-in-comments
source-hash: ff36ccbfbebe976e7f4dfbbded39a9c46a32cbcbfc55a998bbf90199fd2ad28c
bundled: 2026-08-01
title: Links in comments
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
---

# Links in comments

Any comment that contains a URL or cross-reference must use JSDoc (`/** */`), not `//`. This keeps
links consistently findable and IDE-navigable.

**`@see` for standalone references** (external URLs, internal file paths, issue trackers). Always on
its own line:

```ts
/**
 * Validates JWT tokens per RFC 7519.
 *
 * @see https://datatracker.ietf.org/doc/html/rfc7519
 */
```

**`{@link}` for inline references** within prose (code symbols, related functions/types):

```ts
/**
 * Uses {@link validateToken} before processing the request.
 *
 * @param token - Raw JWT string
 * @returns Decoded payload or null if invalid
 */
```

**Do NOT combine them.** `@see {@link ...}` is redundant.

**Workarounds with upstream links:**

```ts
/**
 * WORKAROUND: upstream library does not handle null responses.
 *
 * @see https://github.com/example/library/issues/1234
 * TODO: Remove once issue is fixed in v3.0
 */
```

**Multiple related links:**

```ts
/**
 * @see https://en.wikipedia.org/wiki/Boyer%E2%80%93Moore*string-search*algorithm - Algorithm overview
 * @see https://github.com/example/impl - Reference implementation
 */
```

## See also

- Code comments (authoring) index
