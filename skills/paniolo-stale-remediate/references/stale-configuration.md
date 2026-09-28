---
source-slug: stale-configuration
source-hash: dbd973ec74e2266b2cf123dcc2c1073f21f710c3f5fae30d8fa6f3844dab8fcc
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
- [Enablement And Local Overlay](#enablement-and-local-overlay)
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
| `enabled` | boolean | `true` | Gates `scan`, `run`, `propose`, and `worker` for this selected config; inspection and explicit maintenance remain available |
| `ledgerPath` | string | `.paniolo/staleness` | Repo-relative durable ledger directory. Required when a `staleness` section is present — it has no per-field default. Validated: non-empty, relative, no `.`/`..`/drive-prefix components |
| `agentProfiles` | map: name → `{adapter, model}` | `{default: {adapter: codex, model: default}}` | Required when a `staleness` section is present. Named provider/model pairs; `adapter` must be `codex`, `claude`, or `cursor`; both fields must be non-empty |
| `roles` | `{verifier, verdictChallenger, remediator, patchChallenger}` | every role → `default` | Each value names an `agentProfiles` key; unknown names fail validation |
| `limits` | budget object | see [Limits And Retrieval](#limits-and-retrieval) | Phase, process, packet, repair, and total-invocation budgets |
| `retrieval` | active and shadow retrieval object | active `4` / `50`; shadow off | `topK` and `maxImmediate` bound active scheduling. Optional `shadow` records non-authoritative qmd funnels; see [Limits And Retrieval](#limits-and-retrieval) |
| `surfaces` | `{wikiPages, docs, codeComments}` | empty (admit all) | See [Content Surfaces](#content-surfaces) |
| `autoMerge` | boolean | `false` | Request GitHub auto-merge only after the local merge gate authorizes the exact proposed head |

---

<a id="enablement-and-local-overlay"></a>

## Enablement And Local Overlay

`enabled: false` makes `scan`, `run`, `propose`, and `worker` return a
successful JSON response with `"status": "disabled"` before the ledger
opens. It does not disable inspection, queue maintenance, calibration, or an
independent workflow that passes a different `--config`. See
[stale-triggers](./stale-triggers.md) for the command and trigger matrix.

The canonical `paniolo.config.json` may have a sibling
`paniolo.config.local.json`. The local file recursively overlays the tracked
file: nested objects preserve unspecified tracked values and arrays replace
rather than append. The local file is machine-owned and should be gitignored.

For a private owner override:

```json
{
  "staleness": {
    "enabled": true
  }
}
```

The overlay is loaded only when the selected file is canonically named
`paniolo.config.json`. An explicit differently named file, such as
`.github/staleness-advisory.json`, is independent and does not consume the
local overlay.

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

| Limit | Default | Meaning |
| --- | --- | --- |
| `maxAllegationsPerPhase` | `25` | Allegations processed in each phase |
| `timeoutMs` | `120000` | Wall-clock timeout per agent invocation |
| `maxOutputBytes` | `262144` | Maximum normalized agent output |
| `maxRepairAttempts` | `1` | Validation-repair retries after the first role invocation; zero is allowed |
| `maxPacketBytes` | `49152` | Maximum serialized evidence packet per role invocation |
| `maxInvocationsPerAllegation` | `8` | Hard ceiling across all four roles and their repairs |

`maxAllegationsPerPhase`, `timeoutMs`, `maxOutputBytes`,
`maxPacketBytes`, and `maxInvocationsPerAllegation` must be greater than
zero. The four roles require
`(maxRepairAttempts + 1) * 4 <= maxInvocationsPerAllegation`; otherwise the
configured repair policy could exceed its own hard ceiling and validation
fails.

### Live qmd shadow measurement

`retrieval.shadow` controls continuous measurement of fuzzy qmd retrieval
during ordinary `scan` and `worker` runs:

| Field | Default | Meaning |
| --- | --- | --- |
| `enabled` | `false` | Record qmd candidate funnels without creating allegations or invoking agents |
| `documentPoolSize` | `20` | Maximum configured wiki or docs files retained after fusion per changed entity |
| `sectionTopK` | `2` | Reranked sections marked as work that would be scheduled if the lane were admitted |
| `maxEntitiesPerRun` | `25` | Maximum changed entities measured per scan, bounding model work and ledger growth |

The live producer records separate `qmd-fusion`, `qmd-section-pool`, and
`qmd-rerank` observations. It fingerprints the normalized query, query
template, producer, and exact embedding and reranking models. The reranker
can see only sections inside the retrieved document pool.

This lane is measurement only. It cannot create an allegation, set a
disposition, call an agent, or authorize a merge. A qmd failure appears in
the command's `qmd_shadow` report while deterministic detection continues.
Dry runs write no shadow records.

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
- Every positive-only `limits` value, both active `retrieval` values, and
  all three `retrieval.shadow` budgets must be greater than zero;
  `maxRepairAttempts` may be zero.
- Four times `maxRepairAttempts + 1` may not exceed
  `maxInvocationsPerAllegation`.
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
  "enabled": false,
  "ledgerPath": ".paniolo/staleness",
  "autoMerge": true,
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
    "maxOutputBytes": 262144,
    "maxRepairAttempts": 1,
    "maxPacketBytes": 49152,
    "maxInvocationsPerAllegation": 8
  },
  "retrieval": {
    "topK": 4,
    "maxImmediate": 50,
    "shadow": {
      "enabled": true,
      "documentPoolSize": 20,
      "sectionTopK": 2,
      "maxEntitiesPerRun": 25
    }
  },
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
