---
source-slug: markdown-required-elements
source-hash: 76a83e11ebc1cfb40111b5c4cf73951326ebe0e7e10d62a7c7c21ae7a8938230
bundled: 2026-10-17
title: Required Elements
type: concept
tags:
- authoring
- docs
- markdown
- formatting
updated: 2026-06-18
---

# Required Elements

Every documentation file should include:

1. **Title (H1)** — One per document, at the top
2. **Brief introduction** — 1-3 sentences explaining purpose and scope
3. **Table of Contents** — For docs longer than ~150 lines or with 3+ sections
4. **Body content** — Organized with H2/H3 headers
5. **Optional "See also" section** — Links to related docs (at bottom)

**Retrieval (qmd):** the H1 is what qmd indexes as the document Title and embeds with _every_ chunk (`title: {title} | text: {content}`), so make it specific and keyword-rich — not `Overview` or `Notes`. A vague title weakens semantic matching for the whole file.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
