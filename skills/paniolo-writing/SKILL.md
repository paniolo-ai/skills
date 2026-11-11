---
name: paniolo-writing
description: |
  Writing style standards for communication and prose: clarity, conciseness, audience calibration, tone, active voice, and jargon handling. Use when writing PR descriptions, commit messages, issue summaries, user replies, code comments, or documentation prose. Do NOT use for document formatting/layout (file naming, ToC, anchors) — load paniolo-doc for that.
license: MIT
metadata:
  version: 0.2.0
tags:
- writing
- communication
- style
user-invocable: true
references:
- references/active-vs-passive-voice.md
- references/audience-and-tone.md
- references/avoid-jargon-unless-necessary.md
- references/writing-style-clarity-and-conciseness.md
- references/writing-style-writing-style.md
---

**Requires:** file-read. No terminal needed.

# Writing Style Standards

## Use When

Use this skill when:

- Writing or reviewing PR descriptions, commit messages, and release notes.
- Drafting user replies, issue comments, design reviews, or incident post-mortems.
- Writing prose within documentation, tutorials, or in-code comments.
- Auditing prose for AI fluff, passive voice, or condescending tone.

## Core Writing Rules

### 1. Clarity and Conciseness

- **Write for comprehension, not word count**: Every sentence should convey new information. Cut filler phrases that exist only to pad text.
- **Short sentences**: Aim for <25 words per sentence.
- **Break down complex structures**: When explaining multi-step flows, requirements, or trade-offs, break them into bulleted or numbered lists rather than dense narrative paragraphs.
- **Example**:
  - *Verbose:* "When you are defining props for a component and you need to accept any kind of renderable content, you should make sure that you use the ReactNode type."
  - *Concise:* "Use `ReactNode` for props that accept any renderable content."

### 2. Audience Calibration and Tone

- **Assume a competent engineer**: Skip language fundamentals TypeScript or web developers already know (e.g. do not explain what a boolean or promise is). Focus on project-specific rules, invariants, and architecture decisions.
- **Avoid condescending fillers**: Never use `"obviously"`, `"simply"`, `"just"`, `"clearly"`, or `"as anyone knows"`. State facts directly.
- **Explain the "why"**: When a rule, constraint, or lint flag appears arbitrary, briefly state the failure mode or rationale behind it.

### 3. Active vs. Passive Voice

- **Prefer active voice**: Active voice is direct, unambiguous, and makes the actor clear.
- **Use passive voice only when the actor is unknown or irrelevant**:
  - *Active (preferred):* "The parser rejects trailing commas in JSON config files."
  - *Passive (avoid):* "Trailing commas are rejected by the parser in JSON config files."

### 4. Terminology, Jargon, and Acronyms

- **Define project-specific terms on first use**: If a pattern or concept is specific to this codebase (e.g. warm slots, meta-harness, tagged errors), provide a one-line definition upon first introduction.
- **Spell out non-universal acronyms**: Write `"AST (Abstract Syntax Tree)"` on first mention before using the acronym throughout the text.
- **Prefer plain language over buzzwords**: State concrete behavior rather than abstract marketing phrases.

## Quick Review Decision Tree

```text
Is this prose unclear, dense, or awkward?
├── Too many words or complex clauses → Shorten; split into bulleted list if >2 ideas
├── Passive construction              → Rewrite with the subject performing the action
├── Unexplained jargon or acronym     → Add brief inline definition on first mention
└── Condescending or patronizing word → Remove ("simply", "obviously", "just")
```

## Evaluations (I/O Examples)

**Input:** "Draft a PR description for our worktree pool cleanup command"
**Expected:** Agent outputs a concise, structured description with active voice, bulleted key changes, rationale, and verification commands with zero condescending filler.

**Input:** "Review this user reply for clarity and tone"
**Expected:** Agent audits the passage, flags wordy phrasing, rewrites passive sentences into active voice, and removes patronizing qualifiers.

**Input:** "Format this markdown file with ToC and anchor tags"
**Expected:** Agent hands off to [paniolo-doc](../paniolo-doc/SKILL.md) for document structure and formatting.

## Do Not

- Do not use for markdown document layout, file naming, ToC, or anchor tags — load `paniolo-doc`.
- Do not fabricate facts, statistics, or citations.
- Do not write verbose, passive, or patronizing prose.
- Do not strip technical substance in pursuit of brevity; conciseness means high signal-to-noise ratio.

## References

- [writing-style-writing-style](references/writing-style-writing-style.md)
- [active-vs-passive-voice](references/active-vs-passive-voice.md)
- [audience-and-tone](references/audience-and-tone.md)
- [avoid-jargon-unless-necessary](references/avoid-jargon-unless-necessary.md)
- [writing-style-clarity-and-conciseness](references/writing-style-clarity-and-conciseness.md)
- [paniolo-doc](../paniolo-doc/SKILL.md)

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
