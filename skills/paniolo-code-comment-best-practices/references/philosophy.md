---
source-wiki: sharp-shooter-wiki
source-slug: philosophy
source-hash: 1d3f1bc80899ea0292c155f106bd5ae106a1369f223ba8315af7e6316c4118d0
bundled: 2026-08-01
title: Philosophy
type: concept
tags:
- authoring
- code-comments
updated: 2026-06-18
---

# Philosophy

**Explain the "why," not the "what."** The code is the truth; the comment is the reason. Avoid
redundant descriptions like `// set age to 32`.

**Value-add only.** If the code is already simple and obvious, do not add a comment.

**Future self.** Provide the context needed to understand decisions six months from now — why a
specific workaround was used, what constraint forced a particular shape.

**Documentation vs. clarification:**

- **JSDoc (`/** */`)\*\* — for *consumers\* of the code. Describes the API, inputs, and outputs so
  they don't have to read the implementation.
- **Inline (`//`)** — for _maintainers_. Explains the "why" behind non-obvious logic or performance
  trade-offs.

**No hazard lights.** A comment warning readers away from bad code is an admission of guilt. If code
is complex, explain the _technical constraint_ that forced it, rather than apologizing for the
complexity. If a comment requires a long paragraph, the code is likely too complex and should be
refactored.

**Professionalism.** Never use comments to vent frustration, blame others, or use unprofessional
language.

## See also

- Code comments (authoring) index
