---
source-wiki: sharp-shooter-wiki
source-slug: anti-patterns
source-hash: dc05cf3d28287b12bc22814433c96c0481ca03a6712018793b65277b2015a513
bundled: 2026-08-01
title: Anti-patterns
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
---

# Anti-patterns

**❌ Commented-out code** — use git history instead; delete dead code.

**❌ Version history in comments** — use git log instead.

**❌ Author attribution** — use git blame instead.

**❌ Over-commenting every line** — makes code harder to read, not easier.

**❌ Apologizing or explaining bad code** — refactor instead. If complexity is forced by a real
constraint, explain the constraint specifically; don't just say "sorry this is messy."

**❌ Jokes, easter eggs, or passive-aggressive comments** — keep comments professional and
constructive. Never vent frustration or blame others.

**❌ Stale TODOs with no context** — review during planning; remove TODOs that have been superseded
or are no longer actionable.

## See also

- Code comments (authoring) index