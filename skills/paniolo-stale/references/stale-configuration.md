---
source-slug: stale-configuration
source-hash: 5904e2969b09634bde76e2abc29194963cbc0b990b96ac965218349e8b43a4ff
bundled: 2026-09-27
title: Stale Configuration
type: concept
tags:
- staleness
- harness-eng
- configuration
updated: 2026-09-27
---

# Stale Configuration

The `staleness` section of `paniolo.config.json` is the deployment control
plane for `paniolo stale`. Per
decision-staleness-configuration-authority, configuration
owns deployment policy — model fleets, role assignments, budgets, ledger
placement, content scopes — while provider protocols and safety invariants
remain compiled Rust that configuration cannot relax.

`paniolo stale` reads the section at invocation time: `--config` names the
file (default `paniolo.config.json` under `--root`), and the `staleness`
key inside it is validated as a whole. An absent section selects defaults;
`null` does too. A malformed section is an error, not a warning.

## Contents

- [Schema](#schema)
- [Agent Profiles And Roles](#agent-profiles-and-roles)
- [Limits And Retrieval](#limits-and-retrieval)
- [Content Surfaces](#content-surfaces)
- [Validation Rules](#validation-rules)
- [CLI Override Precedence](#cli-override-precedence)
- [Example](#example)
- [See Also](#see-also)

---

<a id="schema"></a>

## Schema

All keys are camelCase.

| Field | Type | Default | Notes |
| --- | --- | --- | --- |
| `ledgerPath` | string | `.paniolo/staleness` | Repo-relative durable ledger directory. Required when a `staleness` section is present — it has no per-field default. Validated: non-empty, relative, no `.`/`..`/drive-prefix components |
| `agentProfiles` | map: name → `{adapter, model}` | `{default: {adapter: codex, model: default}}` | Named provider/model pairs. `adapter` must be `codex`, `claude`, or `cursor`; both fields must be non-empty |
| `roles` | `{verifier, verdictChallenger, remediator, patchChallenger}` | every role → `default` | Each value names an `agentProfiles` key; unknown names fail validation |
| `limits` | `{maxAllegationsPerPhase, timeoutMs, maxOutputBytes}` | `25` / `120000` / `262144` (256 KiB) | Per-phase work cap, per-invocation wall-clock timeout (ms), max normalized agent output bytes. All must be greater than zero |
| `retrieval` | `{topK, maxImmediate}` | `4` / `50` | `topK` candidates per changed entity schedule immediately; `maxImmediate` caps materialized allegations per scan — overflow persists as deferred candidates in the `R-` run record. Both must be greater than zero |
| `surfaces` | `{wikiPages, docs, codeComments}` | empty (admit all) | See [Content Surfaces](#content-surfaces) |
| `autoMerge` | boolean | `false` | Request GitHub auto-merge only after the local merge gate authorizes the exact proposed head |

---

<a id="agent-profiles-and-roles"></a>

## Agent Profiles And Roles

`agentProfiles` decouples "which CLI + model" from "which role uses it".
A profile is `{adapter, model}`; a role is assigned a profile name.

```json
"agentProfiles": {
  "codex-gpt-5-5": { "adapter": "codex", "model": "gpt-5.5" },
  "cursor-swe-2-high": { "adapter": "cursor", "model": "swe-2-high" }
},
"roles": {
  "verifier": "codex-gpt-5-5",
  "verdictChallenger": "codex-gpt-5-5",
  "remediator": "cursor-swe-2-high",
  "patchChallenger": "codex-gpt-5-5"
}
```

Pairing a verifier family with a *different* remediator family keeps the
challengers auditing across vendors rather than agreeing with themselves.
The `devin` adapter exists in code but is not admitted by the CLI's adapter
whitelist — naming it is an error.

---

<a id="limits-and-retrieval"></a>

## Limits And Retrieval

`limits` are operational budgets, not safety semantics: they cap spend and
runaway work without weakening any gate. `retrieval` bounds the candidate
pool a scan materializes — overflow is recorded as `deferred`, never
dismissed, so tightening `maxImmediate` delays work instead of dropping it.

---

<a id="content-surfaces"></a>

## Content Surfaces

Each of `surfaces.wikiPages`, `surfaces.docs`, and `surfaces.codeComments`
takes:

| Field | Meaning |
| --- | --- |
| `repositories` | Workspace repo keys admitted to this surface; empty admits all |
| `include` | Repo-relative glob patterns; empty includes everything |
| `exclude` | Repo-relative glob patterns subtracted from the include set |

Patterns must be valid globs, repo-relative: no leading `/`, no `:`, no
`..` segments. Backslashes are normalized to `/` before validation.

---

<a id="validation-rules"></a>

## Validation Rules

The whole block is validated on load; any violation aborts the command:

- `ledgerPath` must be a non-empty repo-relative path without parent
  traversal or a drive prefix.
- Every `limits` and `retrieval` value must be greater than zero.
- `surfaces.*.repositories` may not contain blank keys; `include`/`exclude`
  must be valid repo-relative globs.
- Every profile needs a non-empty `adapter` and `model`.
- Every `roles.*` value must reference a declared `agentProfiles` key.

The legacy fallback applies only without `--config` and without a
`staleness` section: if `<root>/staleness/` exists while the configured
ledger path does not, the ledger resolves to `staleness/` for that run.

---

<a id="cli-override-precedence"></a>

## CLI Override Precedence

Role and budget flags on `run` and `worker` are one-run overrides with
higher precedence than config, recorded in invocation evidence; they never
silently rewrite the saved configuration.

- `--adapter` / `--model` override the verifier profile; challenger,
  remediator, and patch-challenger fall back to them when their own flags
  are absent.
- `--challenger` / `--challenger-model`, `--remediator` /
  `--remediator-model`, `--patch-challenger` / `--patch-challenger-model`
  override per role.
- `--max`, `--timeout-ms`, `--max-output-bytes` override `limits`; all must
  be greater than zero.

---

<a id="example"></a>

## Example

The harness workspace runs with two profiles — a Codex verifier/challenger
family and a Cursor remediator — and scoped surfaces:

```json
"staleness": {
  "ledgerPath": ".paniolo/staleness",
  "agentProfiles": {
    "codex-gpt-5-5": { "adapter": "codex", "model": "gpt-5.5" },
    "cursor-swe-2-high": { "adapter": "cursor", "model": "swe-2-high" }
  },
  "roles": {
    "verifier": "codex-gpt-5-5",
    "verdictChallenger": "codex-gpt-5-5",
    "remediator": "cursor-swe-2-high",
    "patchChallenger": "codex-gpt-5-5"
  },
  "limits": {
    "maxAllegationsPerPhase": 25,
    "timeoutMs": 120000,
    "maxOutputBytes": 262144
  },
  "retrieval": { "topK": 4, "maxImmediate": 50 },
  "surfaces": {
    "wikiPages": {
      "repositories": ["paniolo-wiki", "sharp-shooter-wiki"],
      "include": ["wiki/**/*.md"]
    },
    "docs": {
      "repositories": ["ranch-hand"],
      "include": ["README.md", "CONTRIBUTING.md", "docs/**/*.md"],
      "exclude": ["target/**", "vendor/**", "generated/**"]
    },
    "codeComments": {
      "repositories": ["ranch-hand"],
      "include": ["**/*.rs", "**/*.ts", "**/*.tsx"],
      "exclude": ["target/**", "vendor/**", "generated/**"]
    }
  }
}
```

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
