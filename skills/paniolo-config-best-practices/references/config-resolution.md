---
source-slug: paniolo-config-resolution
source-hash: e8d0850c23c9d84e7ed9e53d0280e21acab11c54e4e7d8c24fa8dcfa402aa259
bundled: 2026-10-16
title: Paniolo Config Resolution
type: concept
tags:
- harness-eng
- configuration
- paniolo-config
updated: 2026-10-12
---

# Paniolo Config Resolution

A Paniolo setting cannot be read off one file. The value a command uses is
resolved from two files and several layers. Some of that resolution is
inspectable with a command; the rest is not, and knowing which is which is
the whole skill.

The merge and layering rules themselves are canonical in
paniolo-config-schema. This page covers only what the tooling will and
will not tell you, which that page does not.

## Ask the tool first, within its surface

```bash
paniolo evolve config effective [--target <repo>] [--json]
paniolo evolve config graph [--json]
```

`effective` resolves **one scan invocation** through built-in, preset, host
or node, and CLI layers. It reports `profile`, `target`, selected `path` and
availability, `failOn`, `harnesses`, and every `rules.*` entry, each tagged
with where it came from. `graph` reports the normalized repository graph and
its provenance. Both are read-only.

It lives under `evolve` because `evolve` owns writing harness files; `scan`
is read-only by contract and does not grow a config surface.

## What it does not cover

**Only the scan surface.** `staleness`, `qmd`, `wiki`, `skill`, `agent`, and
`adapter` have no effective view. A question about any of them cannot be
answered by this command, and its silence is not evidence that a section is
unset.

**Not which file a value came from.** Provenance is layer-level — `builtin`,
`preset`, `defaults`, `type`, `host`, `node`, `cli`. There is no variant for
the gitignored `paniolo.config.local.json`, because `Config::load` merges the
overlay into the document *before* layering runs. A value supplied by the
overlay prints as `host`, indistinguishable from the tracked file.

So for anything outside the scan surface, and for any question of the form
"is this value local or shared?", the command cannot answer and you must
resolve by hand.

## Resolving by hand

1. Read `paniolo.config.json`.
2. Read `paniolo.config.local.json` **beside it**. It is gitignored, so
   nothing in the repository reveals it; its absence is a fact to confirm.
3. Merge: objects merge recursively; arrays, scalars, and `null` replace.
4. For a per-repository value, layer the node's own `repos` entry over its
   selected `repoTypes` entry over shared `repoDefaults`.
5. Name the file the answer came from. "The tracked config says X" and "the
   effective value is X" are different claims.

## Traps

- **Quoting the tracked file as the effective state.** The overlay is
  gitignored, so nothing in the repository hints it exists. It can invert the
  switch you are reporting.
- **Treating a clean `evolve config effective` as a whole answer.** It
  answers about scan. It says nothing about the section you may be asking
  about.
- **Reading a mode as a boolean.** A setting can be present and mean neither
  on nor off — a lane configured to file work without ever draining it, for
  instance. Read the variant's own documentation.
- **Editing the tracked file to change local behaviour**, which ships a
  personal preference as team policy; or **editing the overlay to change
  shared behaviour**, which reaches no other machine.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
