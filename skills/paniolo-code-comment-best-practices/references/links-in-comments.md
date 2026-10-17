---
source-slug: links-in-comments
source-hash: a3cd3dbe13fb0175e63623c93b0bb3d439d47a5e1f12494d18b0db3d13890262
bundled: 2026-10-17
title: Links in comments
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
---

# Links in comments

Any comment that contains a URL or cross-reference must use JSDoc (`/** */`), not `//`. This keeps links consistently findable and IDE-navigable.

**`@see` for standalone references** (external URLs, internal file paths, issue trackers). Always on its own line:

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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
