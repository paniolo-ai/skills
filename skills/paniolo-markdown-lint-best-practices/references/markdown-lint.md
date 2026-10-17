---
source-slug: markdown-lint
source-hash: 13a5e2cde84d9f03cb926806a44f06150e207248ab6a143f3e80f787d2de5de8
bundled: 2026-10-16
title: Authoring — Markdown lint
type: index
tags:
- index
- authoring
- markdown-lint
updated: 2026-06-18
---

# Markdown lint (authoring)

Operational reference for markdown lint — loaded from skills and agents.
## Reference

### Heading Anchor Rule

- **What:** A custom `textlint` rule named `require-heading-anchor` enforces an explicit HTML anchor (`

### Table of Contents Rule

- **What:** A custom `textlint` rule named `require-table-of-contents` enforces a `## Table of Contents` section with at least one markdown list item in `docs/**/*.md` when a file exceeds 150 lines or has 3 or more level-2 (`##`) sections.
- **Rollout:** The rule is active now, with a temporary exemption list for legacy docs that still need a ToC added. New or newly touched docs should meet the rule instead of extending the exemption list.
- **Why:** This keeps longer docs scannable, makes AI navigation more reliable, and lines up the lint pipeline with the repo's existing documentation standards.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
