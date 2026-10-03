---
source-slug: markdown-lint
source-hash: 49c9a2f128ade921f28d89e483fdfa6cf6068af2e2f646333fbf992ede75487f
bundled: 2026-10-02
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

- **What:** A custom `textlint` rule named `require-heading-anchor` enforces an
  explicit HTML anchor (`

### Table of Contents Rule

- **What:** A custom `textlint` rule named `require-table-of-contents` enforces
  a `## Table of Contents` section with at least one markdown list item in
  `docs/**/*.md` when a file exceeds 150 lines or has 3 or more level-2 (`##`)
  sections.
- **Rollout:** The rule is active now, with a temporary exemption list for
  legacy docs that still need a ToC added. New or newly touched docs should
  meet the rule instead of extending the exemption list.
- **Why:** This keeps longer docs scannable, makes AI navigation more reliable,
  and lines up the lint pipeline with the repo's existing documentation
  standards.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
