---
name: paniolo-writing-style-best-practices
description: |
  Writing style standards for documentation in this project — clarity, conciseness, audience, tone, active vs passive voice, and jargon handling. Use when writing or reviewing prose in wiki authoring pages, README.md, CONTRIBUTING.md, SKILL.md, or inline code comments. Do NOT use for structural/formatting questions (file naming, ToC, headers, code blocks) — load doc-best-practices instead.
license: MIT
metadata:
  version: 0.1.0
tags:
- writing
references:
- 'wiki: sharp-shooter-wiki:active-vs-passive-voice → references/active-vs-passive-voice.md'
- 'wiki: sharp-shooter-wiki:audience-and-tone → references/audience-and-tone.md'
- 'wiki: sharp-shooter-wiki:avoid-jargon-unless-necessary → references/avoid-jargon-unless-necessary.md'
- 'wiki: sharp-shooter-wiki:doc-content → references/doc-content.md'
- 'wiki: sharp-shooter-wiki:writing-style-clarity-and-conciseness → references/writing-style-clarity-and-conciseness.md'
- 'wiki: sharp-shooter-wiki:writing-style-writing-style → references/writing-style-writing-style.md'
user-invocable: true
---

**Requires:** file-read. No terminal needed.

## Full reference

[writing-style-writing-style](references/writing-style-writing-style.md) —
load on demand for ✅/❌ examples, rationale, and edge cases.

## When invoked

**Preconditions:**

- Read the target file before reviewing or editing prose.

**Clarifying questions:**

- **Defaults (proceed without asking):** assume the task is reviewing prose
  in a doc unless a specific file path or writing task is provided.
- **Always ask:** target file path if reviewing and not provided; topic and
  purpose if writing new prose and not provided. Ask one focused question at
  a time.
- State assumptions when proceeding: "Reviewing prose in the provided
  snippet — let me know if you want a full file review."

**Output format:**

- _Reviewing prose:_ structured report — **Passes** (bullet list) and **Gaps**
  (table: location | issue | suggested fix). Keep it terse.
- _Rewriting prose:_ output only the changed paragraphs/sentences as fenced
  code, with one sentence explaining each change.
- _Question-answering:_ concise prose referencing the relevant doc section.

**Error handling:**

- If the target file is needed but not given and cannot be inferred, ask for
  it before proceeding.
- Do not fabricate content; only rewrite what is provided.

## Quick rules

### Clarity and conciseness

- Write for comprehension, not word count
- Short sentences: aim for <25 words
- Break complex ideas into bulleted lists
- Define acronyms on first use: `"GUI (Graphical User Interface)"`

**✅ Good:**

```markdown
Use `ReactNode` for props that accept any renderable content.
```

**❌ Verbose:**

```markdown
When you are defining props for a component and you need to accept
any kind of renderable content, you should use the `ReactNode` type.
```

**Details:**
[writing-style-clarity-and-conciseness](references/writing-style-clarity-and-conciseness.md)

### Audience and tone

- Assume: competent developer, new to this project
- Skip basics TypeScript developers already know
- Avoid condescending phrases: `"Obviously..."`, `"Simply..."`
- Explain the "why" when rules might seem arbitrary

**Details:** [audience-and-tone](references/audience-and-tone.md)

### Active vs passive voice

- Prefer active voice for clarity and directness
- Use passive only when the actor is unknown or irrelevant

**✅ Active (preferred):**

```markdown
The test fails when the mock returns `undefined`.
```

**❌ Passive (avoid):**

```markdown
The test is failed when `undefined` is returned by the mock.
```

**Details:**
[active-vs-passive-voice](references/active-vs-passive-voice.md)

### Jargon and terminology

- Define technical terms on first introduction
- Especially required for project-specific patterns
- Prefer plain language; add the term once defined

**✅ Good:**

```markdown
Use the non-factory `vi.mock()` pattern (calling `vi.mock()` without
a factory function) for most cases.
```

**❌ Avoid:** `"Use non-factory mocking."` — no definition given.

**Details:**
[avoid-jargon-unless-necessary](references/avoid-jargon-unless-necessary.md)

## Quick decision tree

```text
Is this prose unclear or too long?
├─ Too many words → shorten; break into list if >2 items
├─ Passive voice → rewrite as active
├─ Jargon without definition → add inline definition on first use
└─ Condescending tone → remove qualifier; state fact directly
```

## Evaluations (I/O examples)

**Input:** "Review the prose in this paragraph for writing style"
**Expected:** Agent reads the passage, outputs **Passes** / **Gaps** report
citing specific sentences with suggested rewrites. No structural feedback
(that belongs in `doc-best-practices`).

**Input:** "Rewrite this sentence in active voice"
**Expected:** Agent outputs only the rewritten sentence in a fenced block with
one line of explanation.

**Input:** "Is this tone appropriate for the docs audience?"
**Expected:** Agent references the audience definition (competent developer,
new to project) and gives a concrete yes/no with specific examples.

## Do Not

- Do not use for structural/formatting questions (file naming, ToC, headers, code blocks).
  Load `doc-best-practices` instead.
- Do not fabricate content; only rewrite what is provided.
- Do not give structural feedback when the task is prose review — scope to writing style only.
- Do not rewrite content outside what the user provided or requested.

## References

- Full reference: [writing-style-writing-style](references/writing-style-writing-style.md)
- Structural standards: [doc-content](references/doc-content.md)
- Doc structure skill: `/.agents/skills/doc-best-practices/SKILL.md`