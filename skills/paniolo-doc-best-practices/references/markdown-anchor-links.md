---
source-slug: markdown-anchor-links
source-hash: 6c1aaa5fee68a1e044eb886fb07f8f0e6b1271eb3416371804e40c56d85b0ce8
bundled: 2026-10-17
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

**Rationale:** Explicit anchors ensure consistency across Markdown renderers. GitHub auto-generates anchors, but they can change if heading text is modified. Explicit anchors are stable and renderer-agnostic.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
