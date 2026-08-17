---
name: paniolo-code-comment-best-practices
description: |
  Code comment conventions for TypeScript/React — JSDoc vs inline, when to explain why vs what, params/returns rules, spacing, placement, test comment rules, links in comments (@see vs {@link}), TODO/FIXME format, anti-patterns, writing style. Use when adding, updating, or reviewing comments in .ts or .tsx files. Do NOT use for general documentation questions unrelated to in-code comments.
license: MIT
metadata:
  version: 0.1.0
tags:
- code-comments
- documentation
references:
- 'wiki: sharp-shooter-wiki:anti-patterns → references/anti-patterns.md'
- 'wiki: sharp-shooter-wiki:code-comments → references/code-comments.md'
- 'wiki: sharp-shooter-wiki:constants → references/constants.md'
- 'wiki: sharp-shooter-wiki:inline-comments → references/inline-comments.md'
- 'wiki: sharp-shooter-wiki:jsdoc-formatting → references/jsdoc-formatting.md'
- 'wiki: sharp-shooter-wiki:jsdoc-params-returns → references/jsdoc-params-returns.md'
- 'wiki: sharp-shooter-wiki:jsdoc-when → references/jsdoc-when.md'
- 'wiki: sharp-shooter-wiki:links-in-comments → references/links-in-comments.md'
- 'wiki: sharp-shooter-wiki:philosophy → references/philosophy.md'
- 'wiki: sharp-shooter-wiki:spacing-placement → references/spacing-placement.md'
- 'wiki: sharp-shooter-wiki:test-comments → references/test-comments.md'
- 'wiki: sharp-shooter-wiki:what-not-to-comment → references/what-not-to-comment.md'
- 'wiki: sharp-shooter-wiki:writing-style → references/writing-style.md'
---

**Requires:** file-read. No terminal needed unless validating after edits.

**Depends on:**
[`paniolo-typescript-best-practices/SKILL.md`](../paniolo-typescript-best-practices/SKILL.md) —
load when comment changes are driven by API clarity or type changes.

## Full reference

[code-comments](references/code-comments.md) —
load on demand for detailed formatting patterns, examples, and edge cases.

## When invoked

**Preconditions:**

- Read the file being commented before making changes.
- Check rules for repo-wide constraints.

**Clarifying questions:**

- **Defaults (proceed without asking):** add/update comments in the file already open; follow all
  rules below.
- **Always ask:** which file if not specified.

**Output format:**

- Write changes directly; after edits output a brief bullet list of what was added/changed and why.
- For question-answering: concise prose with inline code, referencing the relevant doc section.

**Error handling:**

- If lint or tsc fails after comment edits, report verbatim and fix before declaring success.

## Quick rules

Core philosophy: explain the "why," not the "what."
([philosophy](references/philosophy.md))

- **JSDoc (`/** \*/`)\*\* for exported functions, components, and types.
  ([jsdoc-when](references/jsdoc-when.md))
- **JSDoc requirement:** Every exported function must have JSDoc. Additionally, any non-trivial
  internal function (complex logic, side effects, branching, or >~3 lines) should include concise
  JSDoc. Prefer documenting intent and side-effects over implementation details.
- **Default to JSDoc for named functions you add** — not just exports. Helper functions, local
  utility functions, test helpers, and hook-internal handlers should usually get concise JSDoc
  unless the name and body are truly trivial. This repo prefers documenting function purpose
  proactively rather than only at module boundaries.
- **Tests and strict-typing helpers count too** — named helpers such as
  `installMockBody`, `makeProps`, `LocationProbe`, `noop`, `noopAsync`,
  `noopUnsubscribe`, and local type guards added to satisfy strict typing should
  get JSDoc with `@param` and `@returns`, even when the body is short.
- **`//`** for logic blocks (`useEffect`, complex conditionals), test descriptions, and grouped
  constants. ([inline-comments](references/inline-comments.md))
- **No types in JSDoc** — no `{Type}` on `@param` / `@returns` in `.ts` / `.tsx`. Use
  `@param name - description` and prose-only `@returns` (or `@returns void`).
  ([jsdoc-formatting](references/jsdoc-formatting.md))
- **Always `@returns`** — write `@returns void` for void functions.
  ([jsdoc-params-returns](references/jsdoc-params-returns.md))
- **Props: document fields directly** — no `@param props` wrapper; list each destructured field.
  ([jsdoc-params-returns](references/jsdoc-params-returns.md))
- **Multi-line JSDoc:** `/**` and `*/` each on their own line.
  ([jsdoc-formatting](references/jsdoc-formatting.md))
- **One blank line above** a JSDoc block; no blank lines between JSDoc and its symbol.
  ([spacing-placement](references/spacing-placement.md))
- **No JSDoc above `describe`/`it`/`test`** — use `//` only when the name isn't self-explanatory.
  ([test-comments](references/test-comments.md))
- **Grouped constants** — use `//` above the group, not JSDoc spanning multiple symbols.
  ([constants](references/constants.md))
- **Symbol comments:** If a comment documents a specific symbol (function, constant, type, class, or
  exported value), use JSDoc (`/** ... */`) rather than `//`. Reserve `//` for inline implementation
  notes and small logic comments. This keeps symbol-level documentation consistent and
  machine-readable.
- **Avoid repeating the symbol name:** The first sentence of a symbol's JSDoc should be a concise
  verb phrase describing its purpose (e.g., "Return the user's profile"), not a restatement of the
  symbol name (avoid: "User profile: returns..." or "Noise filter: ..."). This keeps inline
  documentation focused and avoids redundancy.
- **Max 100 chars per line.**
  ([jsdoc-formatting](references/jsdoc-formatting.md))
- **JSDoc for any comment with a URL** — use `@see` for standalone links (URLs, file paths),
  `{@link}` for inline symbol refs; never `//` for comments containing links.
  ([links-in-comments](references/links-in-comments.md))
- **TODO/FIXME must include context** — `// TODO: [action] — [reason or ticket]`; switch to JSDoc
  when the comment includes a link. ([inline-comments](references/inline-comments.md))
- **No redundant comments** — don't comment obvious code, don't repeat types, don't apologize for
  messy code (refactor instead). ([what-not-to-comment](references/what-not-to-comment.md))
- **Writing style** — full sentences + periods for JSDoc; fragments ok for `//`; active voice;
  present tense; one space after `//`. ([writing-style](references/writing-style.md))
- **Anti-patterns** — no commented-out code, no version history, no author attribution, no jokes or
  passive-aggressive remarks.
  ([anti-patterns](references/anti-patterns.md))

## Validation

```bash
<your lint command>
```

## Do not

- Do not violate repo-wide rules in rules.
- Do not change logic — comments only.
- Do not add types in JSDoc for `.ts`/`.tsx` files.
- Do not expand scope beyond the requested task without calling it out.

## Skill handoffs

- For TypeScript API clarity issues, also load `paniolo-typescript-best-practices`.
- For React hook or component comments, also load `react-best-practices`.

## References

- Full reference: [code-comments](references/code-comments.md)
- Repo-wide rules: rules
- [paniolo-typescript-best-practices/SKILL.md](../paniolo-typescript-best-practices/SKILL.md)
- [react-best-practices/SKILL.md](../paniolo-react-best-practices/SKILL.md)

## Evaluations (I/O examples)

**Input:** "Add JSDoc to all exported functions in `src/utils/format.ts`"
**Expected:** Agent reads the file, adds JSDoc with correct param/returns rules (no types,
destructured props, `@returns void` where applicable), runs lint + tsc, reports what was added.

**Input:** "How should I comment a `useEffect` block?"
**Expected:** Agent answers in prose: use `//` above the `useEffect`, explain the why not the what,
never on the same line. References [inline-comments](references/inline-comments.md).

**Input:** "Add comments" (no file specified)
**Expected:** Agent asks which file before proceeding.
