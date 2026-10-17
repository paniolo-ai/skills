---
source-slug: markdown-file-naming
source-hash: 39b8a88584696a656beb33ef8183400193da8c1246ed113359029d1e48de4028
bundled: 2026-10-17
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
