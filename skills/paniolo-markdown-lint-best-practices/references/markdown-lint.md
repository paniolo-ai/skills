---
source-wiki: sharp-shooter-wiki
source-slug: markdown-lint
source-hash: 4c8c5e23a5274986cdbac1c97004e97609197af512e914bf00025b1b676b91bc
bundled: 2026-07-24
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
Repo hub: `docs/repos/llm-wiki/markdown-lint-best-practices.md` (thin router during wiki migration).

## Pages

- AI Agents: First-Pass Markdown
- Auto-fixing mechanical issues
- Internal Link Validation
- Link Style and Conventions
- Link Validation (`remark`)
- Links (remark)
- Markdownlint and Textlint Split
- Anchors (`docs/**/*.md` only)
- Pre-Authoring Checklist
- File size limits (textlint enforced)
- Links
- Structure (markdownlint)
- Size limits
- Structural Consistency (`markdownlint`)
- Structure (markdownlint)
- Text and Content Validation (`textlint`)
- The three linters
- Validation Commands

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

## See also

- Authoring index
- Repo hub: `docs/repos/llm-wiki/markdown-lint-best-practices.md`