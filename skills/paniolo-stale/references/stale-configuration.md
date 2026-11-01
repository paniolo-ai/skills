---
source-slug: stale-configuration
source-hash: 97b0319c5074d928bdf5b8382a7e07aa383508847dba308ac824729ec4f6a8ba
bundled: 2026-11-01
title: Stale Configuration
type: concept
tags:
- staleness
- harness-eng
- configuration
updated: 2026-10-27
---

# Stale Configuration

The `staleness` section of `paniolo.config.json` is the deployment control plane for `paniolo stale`. Per decision-staleness-configuration-authority, configuration owns deployment policy — model fleets, role assignments, budgets, ledger placement, content scopes — while provider protocols and safety invariants remain compiled Rust that configuration cannot relax.

`paniolo stale` reads the section at invocation time: `--config` names the file (default `paniolo.config.json` under `--root`), and the `staleness` key inside it is validated as a whole. An absent section selects defaults; `null` does too. A malformed section is an error, not a warning.

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
| `enabled` | boolean | `true` | Gates `scan`, `run`, `propose`, and `worker`; agent `flag` filing additionally requires an explicit effective `true` |
| `ledgerPath` | string | `.paniolo` | Repo-relative directory for the Git-tracked snapshot and writer lease; the live database lives in native state. Required when a `staleness` section is present. Validated: non-empty, relative, no `.`/`..`/drive-prefix components |
| `ledger.backend` | `sqlite` | `sqlite` | Production commands reject legacy `json`; `stale migrate` can still parse it to import an existing tree |
| `agentProfiles` | map: name → `{adapter, model}` | `{default: {adapter: codex, model: default}}` | Required when a `staleness` section is present. Admitted adapters: `codex`, `claude`, `cursor`, `devin`; both fields must be non-empty |
| `roles` | `{scout?, verifier, verdictChallenger, remediator, patchChallenger}` | scout absent; other roles → `default` | Each value names an `agentProfiles` key; unknown names fail validation. The optional scout requires a CLI release with configurable scout support |
| `limits` | budget object | see [Limits And Retrieval](#limits-and-retrieval) | Phase, process, packet, repair, and total-invocation budgets |
| `retrieval` | active and shadow retrieval object | active `4` / `50`; shadow off | `topK` and `maxImmediate` bound active scheduling. Optional `shadow` records non-authoritative qmd funnels; see [Limits And Retrieval](#limits-and-retrieval) |
| `surfaces` | `{wikiPages, docs, codeComments, htmlDocs}` | empty (admit all) | See [Content Surfaces](#content-surfaces) |
| `autoMerge` | boolean | `false` | Request GitHub auto-merge only after the local merge gate authorizes the exact proposed head |

---

<a id="enablement-and-local-overlay"></a>

## Enablement And Local Overlay

`enabled: false` makes `scan`, `run`, `propose`, and `worker` return a successful JSON response with `"status": "disabled"` before the ledger opens. It does not disable inspection, queue maintenance, calibration, or an independent workflow that passes a different `--config`. See [stale-triggers](./stale-triggers.md) for the command and trigger matrix.

Agent filing through `stale flag` is intentionally stricter: the effective selected config must **explicitly** contain `enabled: true`. The built-in default of `true` does not count. Missing, false, and absent sections return `status: disabled` with no report record. This lets a working agent report evidence it already encountered only when its owner has opted into that behavior; filing never launches a remediation agent.

The canonical `paniolo.config.json` may have a sibling `paniolo.config.local.json`. The local file recursively overlays the tracked file: nested objects preserve unspecified tracked values and arrays replace rather than append. The local file is machine-owned and should be gitignored.

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
  "devin-swe-2-max": { "adapter": "devin", "model": "swe-2-max" },
  "devin-swe-2-high": { "adapter": "devin", "model": "swe-2-high" }
},
"roles": {
  "scout": "devin-swe-2-max",
  "verifier": "devin-swe-2-max",
  "verdictChallenger": "devin-swe-2-high",
  "remediator": "devin-swe-2-max",
  "patchChallenger": "devin-swe-2-high"
}
```

Use independent model profiles for verification and challenge. The frozen Q1
replay passed with Devin Max/High; Max challenging itself had a critical false
dismissal. The admitted adapter names are `codex`, `claude`, `cursor`, and
`devin`. If `roles.scout` is absent, the opt-in merge-scoped scout uses Cursor
Auto for backward compatibility. Older CLI releases ignore `roles.scout` and
still use Cursor Auto, so release the configurable scout before a Devin-only
scout trial.

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
The configured globs select which files are scanned; parser support for a
language does not override an excluded or unselected path.

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

Production commands require an existing, compatible native
`workspace.sqlite`. Neither `ledger.backend=json` nor the retired
`PANIOLO_STALE_JSON` environment override selects the old tree. An
explicit `stale migrate` imports a saved JSON tree; an absent source
fails rather than creating a clean-looking empty ledger.

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

The Devin-only pilot configuration assigns SWE-2 Max to scout, verification,
and remediation, and SWE-2 High to both challengers. The scheduled worker and
remediation auto-merge remain off; guarded ledger-only auto-merge is enabled:

```json
"staleness": {
  "enabled": false,
  "ledgerPath": ".paniolo",
  "autoMerge": false,
  "ledgerAutoMerge": true,
  "agentProfiles": {
    "devin-swe-2-high": { "adapter": "devin", "model": "swe-2-high" },
    "devin-swe-2-max": { "adapter": "devin", "model": "swe-2-max" }
  },
  "roles": {
    "scout": "devin-swe-2-max",
    "verifier": "devin-swe-2-max",
    "verdictChallenger": "devin-swe-2-high",
    "remediator": "devin-swe-2-max",
    "patchChallenger": "devin-swe-2-high"
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
      "repositories": ["ranch-hand", "paniolo.ai"],
      "include": ["**/*.rs", "**/*.ts", "**/*.tsx", "**/*.js"],
      "exclude": ["target/**", "vendor/**", "generated/**", "**/node_modules/**", "**/dist/**", "**/test-results/**", "**/.wrangler/**"]
    },
    "htmlDocs": {
      "repositories": ["paniolo.ai"],
      "include": ["*.html", "docs/**/*.html", "pages/**/*.html"],
      "exclude": ["drafts/**", "react/**", "dist/**", "node_modules/**", "test-results/**"]
    }
  }
}
```

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
