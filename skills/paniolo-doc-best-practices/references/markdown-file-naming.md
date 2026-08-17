---
source-wiki: sharp-shooter-wiki
source-slug: markdown-file-naming
source-hash: ffd1431cb40c600e1c08e1d4311f2b0eaff44cdcb1205985939af5ff6cd7baa4
bundled: 2026-08-01
title: File Naming
type: concept
tags:
- authoring
- docs
- markdown
- formatting
updated: 2026-06-18
---

# File Naming

Use lowercase with hyphens:

**✅ Good:**

```text
typescript-lint-best-practices.md
markdown-lint-best-practices.md
playwright-test-best-practices.md
```

**❌ Bad:**

```text
Doc*Best*Practices.md
UnitTestBestPractices.md
playwright.best.practices.md
```

**Rationale:** Hyphens are the most common URL separator, and Markdown files are often served as web
content (e.g., GitHub). Lowercase avoids case-sensitivity issues across operating systems.

**Retrieval (qmd):** the filename is qmd's Title fallback and part of the indexed path,
so use descriptive, searchable slugs with the real terms an agent would query.

## See also

- Doc structure and formatting (authoring) index
