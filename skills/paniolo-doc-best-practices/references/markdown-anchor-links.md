---
source-wiki: sharp-shooter-wiki
source-slug: markdown-anchor-links
source-hash: c9e1dba8030903b222970ebd67e8c114fc1a120067333779661aefa9370f3ae7
bundled: 2026-08-01
title: Anchor Links
type: concept
tags:
- authoring
- docs
- markdown
- formatting
updated: 2026-06-18
---

# Anchor Links

Use explicit `<a id="...">` tags for all headings that appear in the Table of Contents.

**Format:**

- Convert heading text to lowercase
- Replace spaces with hyphens
- Remove punctuation (except hyphens)
- Strip leading numbers if auto-generated

**Rationale:** Explicit anchors ensure consistency across Markdown renderers. GitHub auto-generates
anchors, but they can change if heading text is modified. Explicit anchors are stable and
renderer-agnostic.

## See also

- Doc structure and formatting (authoring) index