---
name: paniolo-qmd
description: |
  Find relevant skills, docs, and repo-specific guidance with the Paniolo qmd workspace search. Use before task-shaped work, when qmd results mention a repo-local skill, or when maintaining the qmd index. Do not use for editing qmd implementation code; use script, TypeScript, or test guidance for that.
license: MIT
metadata:
  version: 0.4.21
tags:
- qmd
references: []
user-invocable: true
---

# QMD

## Use When

- Discovering which harness or repo-local skill/doc applies to a task.
- Searching across the Paniolo workspace before reading or editing code.
- Re-indexing after adding or changing markdown guidance.
- Checking whether qmd can see a skill, doc, or repo-local instruction file.

---

## Setup

Install dependencies once with `pnpm install`. The harness pulls the correct
`@paniolo/cli-<platform>-<arch>` binary for the current platform through the `@paniolo/cli`
wrapper, so no global qmd install is needed. Set `PANIOLO_BIN` only when using a locally built
binary.

- **Windows (PowerShell):** run `pnpm run qmd:reindex` to build the BM25 index and,
  on a GPU-capable machine, generate the hybrid query embeddings.
- **WSL (bash):** run `pnpm run qmd:reindex` to build the BM25 index. GPU embedding
  stays on the Windows side; WSL uses fast BM25 `search`.

Each platform keeps its own SQLite index on native local disk so Windows and
WSL indexes never collide.

---

## Execution Workflow

1. **Search** for candidates with `pnpm run qmd -- search` (BM25) or
   `pnpm run qmd -- query` (hybrid, GPU on Windows).
2. **Load** only the clearly relevant returned files (or fetch with
   `qmd get`/`multi-get`).
3. If a result points to another repo's `.agents/skills/<name>/SKILL.md`, read that
   repo-local skill for work in that repo.
4. After adding or editing markdown guidance, run `pnpm run qmd -- reindex`.

---

## Pick the right mode

- **`search` (BM25)** — exact words, titles, code symbols, rare phrases. Fast,
  any platform, no model. Reach for this first when you know the term.
- **`query` (hybrid)** — conceptual recall: the task is described indirectly or
  uses different words than the guidance. GPU-accelerated on Windows; on WSL it
  falls back to BM25. Don't overuse semantic search when a keyword would do.

```bash
pnpm run qmd -- search "zustand store hook"
pnpm run qmd -- query  "how do we structure effect-ts error handling"
```

## Author the query yourself — don't lean on expansion

**You are a better query expander than the built-in model.** Prefer a
**structured multi-line query** with your own `intent:`/`lex:`/`vec:`/`hyde:`
fields over a bare sentence.

### Structured query syntax

A structured query uses one or more of the following fields. The first line gets
2x weight, so put the most important signal there.

- `intent:` — what you're trying to find **and what to avoid**. Almost always
  supply this; it steers ranking away from nearby-but-wrong guidance.
- `lex:` — exact terms, skill slugs, file names, code symbols (your own keyword
  expansion).
- `vec:` — the idea paraphrased in natural, source-like language.
- `hyde:` — a hypothetical answer or doc passage (50–100 words) that would satisfy
  the request.

Write at least `intent:` plus one of `lex:`/`vec:`.

```bash
pnpm run qmd -- query $'intent: find the harness vitest setup, not playwright e2e
lex: vitest mock hook test best-practices
vec: how to unit test a react hook with vitest
hyde: The vitest skill explains mocking a hook and asserting state transitions with renderHook.'
```

If you genuinely have only one rare token or a verbatim phrase, that's a job
for `qmd search`, not a bare `qmd query`. Inspect ranking with
`pnpm run qmd -- query --format json --explain "$query"`.

### MCP `query` tool

On Windows (GPU tier), qmd may also be exposed as an MCP server. Prefer the MCP
`query` tool over shelling out when present, and apply the same craft: pass a
structured `searches` array of typed sub-queries plus `intent`, and scope with
the **plural `collections`** array (a singular `collection` is silently ignored):

```json
{
  "searches": [
    { "type": "lex", "query": "vitest mock hook test" },
    { "type": "vec", "query": "how to unit test a react hook with vitest" }
  ],
  "intent": "harness vitest setup, not playwright e2e",
  "collections": ["harness"],
  "limit": 5
}
```

On WSL the MCP server is typically a no-op (BM25 tier) — use the CLI instead.

## Scope when results drift

Searches span every workspace repo by default. Narrow with `-c <repo>` (the
collection name = repo name) when a broad search pulls the wrong corpus:

```bash
pnpm run qmd -- search "auth flow" -c paniolo
pnpm run qmd -- query  "deployment runbook" -c my-app -c paniolo.ai
```

`qmd context list` shows each collection's context tree (root plus `/.agents/skills/`,
`sharp-shooter-wiki/wiki/`, etc.) — those context lines come back with results to help you pick.

## Retrieve sources — by docid, with line ranges

Results carry a `#docid` and a `qmd://` path, and `get`/`multi-get` output is
**line-numbered by default**. Fetch and cite both; read a window with the
`:from:count` suffix instead of piping through `sed`/`head`/`tail`:

```bash
pnpm run qmd -- get "#abc123"                 # whole doc
pnpm run qmd -- get "#abc123:120:40"          # 40 lines from line 120
pnpm run qmd -- multi-get "#abc123,#def456"   # compare several hits
pnpm run qmd -- get "#abc123" --full-path     # on-disk path for Read/Edit
```

Add `--full-path` when you need a path to hand to `Read`/`Edit` or an editor.

---

## Commands

- Search: `pnpm run qmd -- search "<task description>"`
- Hybrid query (GPU on Windows): `pnpm run qmd -- query "<query or structured fields>"`
- Retrieve: `pnpm run qmd -- get <#docid|path[:from:count]>` / `multi-get <glob|list>`
- Discover: `pnpm run qmd -- ls [collection]`, `status`, `context list`
- Re-index: `pnpm run qmd -- reindex`

---

## Key Patterns

- Use the harness launcher instead of calling `qmd` directly so each platform
  gets the correct binary and native-disk index.
- The workspace index is generated from `harness.code-workspace` and spans
  harness plus sibling Paniolo repos.
- Harness skills are shared baseline guidance; repo-local skills are the
  operating procedures for that repo's own files, validators, and workflows.
- `-c <repo>` scoping does not include the harness skill collection. Dot-prefixed
  directories are indexed separately, so `.agents/skills/` is its own collection
  (`harness-agents-skills`). Use an unscoped search to search both a repo and
  its skills.
- qmd failure is not blocking. Inspect `.agents/skills/` and the wiki directories configured in
  `paniolo.config.json` across the workspace directly and continue.

---

## References

- Repo entry point: [AGENTS.md](/AGENTS.md)
- Harness overview: [README.md](/README.md)

## Do Not

- Do not call `qmd` directly when the harness launcher is available.
- Do not paste the user's sentence into a bare `qmd query` — author
  `intent:`/`lex:`/`vec:`/`hyde:` yourself.
- Do not slice retrieved files with `sed`/`head`/`tail` — use the
  `path:from:count` suffix or `--from`/`-l` (output is already line-numbered).
- Do not stop at snippets when you need facts, quotes, or nuance — fetch the
  document with `get`/`multi-get` first.
- Do not copy repo-specific skills into harness only for visibility.
- Do not load every qmd result; choose the smallest useful context set.
- Do not stop task work solely because qmd search or query failed.