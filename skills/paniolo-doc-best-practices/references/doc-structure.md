---
source-wiki: sharp-shooter-wiki
source-slug: doc-structure
source-hash: 159131216d34242ca3bd72ee129ac2d0b14ad88e219734219d472cdf752d92a8
bundled: 2026-07-24
title: Authoring — Doc structure and formatting
type: index
tags:
- index
- authoring
- doc-structure
updated: 2026-06-18
---

# Doc structure and formatting (authoring)

Operational reference for doc structure and formatting — loaded from skills and agents.
Repo hub: `docs/doc-structure-and-formatting.md` (thin router during wiki migration).

## Pages

- Anchor Links
- Code Blocks
- component-props
- Emphasis
- File Length
- File Naming
- Header 2
- Line Length
- Lists
- Required Elements
- Tables

## Reference

### Headers

**Use ATX-style headers (`#`)** — not Setext-style (`===` or `---`):

```markdown
# Header 1
## Header 2
### Header 3
```

Do not skip heading levels (for example `#` followed directly by `###`).

### Table of contents

Example structure:

```markdown
- Section One
  - Subsection A
  - Subsection B
- Section Two
```

### Table of contents placement

Place the ToC immediately after the introduction:

```markdown
# Document Title

Brief introduction explaining what this document covers.

## Table of Contents
...
```

## See also

- Authoring index
- Repo hub: `docs/doc-structure-and-formatting.md`