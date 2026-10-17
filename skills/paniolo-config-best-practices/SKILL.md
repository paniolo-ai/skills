---
name: paniolo-config-best-practices
description: |
  How to read a paniolo.config.json correctly before relying on any value in it — ask `paniolo evolve config effective` for the scan surface, and know that it covers nothing else and never reveals whether a value came from the gitignored paniolo.config.local.json overlay that can invert a setting. Use when asked what a Paniolo setting is, when explaining why a paniolo command behaved unexpectedly, or before quoting any configured value back to a user. Do NOT use to create a config from scratch (paniolo-config-init) or to migrate and deduplicate an existing one (paniolo-config-upgrade).
license: MIT
metadata:
  version: 0.1.0
tags:
- harness
- config
user-invocable: true
references:
- references/config-resolution.md
---

**Requires:** file-read, terminal.

# paniolo-config-best-practices — read a Paniolo config correctly

A Paniolo setting cannot be read off one file. One command resolves part of
it; the rest must be resolved by hand. Quoting `paniolo.config.json` is a
guess that looks like a fact.

## Use When

- Asked what a Paniolo feature is set to, or whether something is enabled.
- Explaining why `paniolo scan`, `stale`, `wiki`, or `qmd` behaved unlike the
  config appears to say.
- About to quote a configured value to a user, or to act on one.

For creating a config use `paniolo-config-init`; for migrating or
deduplicating one use `paniolo-config-upgrade`.

## Execution Workflow

1. **If the question is about scan** — rules, severities, `failOn`, harness
   scope, which path a repo resolves to — ask the tool:

   ```bash
   paniolo evolve config effective [--target <repo>] [--json]
   ```

   It resolves one scan invocation through built-in, preset, host or node,
   and CLI layers, tagging each value with its source. Read-only.

2. **For every other section** — `staleness`, `qmd`, `wiki`, `skill`,
   `agent`, `adapter` — there is no effective view. Resolve by hand:
   read `paniolo.config.json`, then `paniolo.config.local.json` beside it,
   then merge (objects merge recursively; arrays, scalars and `null`
   replace), then layer node over `repoTypes` over `repoDefaults`.

3. **Name the file.** "The tracked config says X" and "the effective value is
   X" are different claims. Say which one you are making.

See [config resolution](references/config-resolution.md) for the coverage boundary, the provenance
gap, and the trap list.

## Key Patterns

- **`evolve config effective` covers scan only.** Its silence about a section
  is not evidence that the section is unset.
- **It never names the file.** Provenance is layer-level (`builtin`, `host`,
  `node`, …). A value supplied by the gitignored overlay prints as `host`,
  indistinguishable from the tracked file.
- **A scalar in the overlay wins outright.** Tracked `false` plus overlay
  `true` means on, and automation gates read the merged value.
- **Arrays replace, never append.** An overlay list discards every entry the
  tracked file declared.
- **An enum is not a boolean.** A setting can be present and mean neither on
  nor off; read the variant's own documentation.
- **The overlay is machine-local preference, never shared policy.**

## References

- [Paniolo Config Resolution](references/config-resolution.md) — what the tooling
  does and does not tell you
- Paniolo Config Schema — canonical merge order and the full section
  reference
- Stale Configuration — the `staleness` section
- `paniolo-config-init` — scaffold a new config
- `paniolo-config-upgrade` — migrate, deduplicate, register repos

## Do Not

- Do not report the tracked file's value as the effective one.
- Do not treat a clean `evolve config effective` as answering a non-scan
  question.
- Do not conclude a feature is off because the tracked file says so.
- Do not assume arrays combine across the two files.
- Do not edit the tracked file to change local behaviour, or the overlay to
  change shared behaviour.
- Do not put secrets in either file.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
