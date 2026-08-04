---
source-wiki: sharp-shooter-wiki
source-slug: forbidden-feature-folder-names
source-hash: 723241326f5bd97d48fd7c1313642ec1eed2845a4249ddf5c4dc354ede5c6c15
bundled: 2026-08-01
title: Forbidden feature-folder names
type: concept
tags:
- authoring
- code-layout
- file-organization
updated: 2026-06-18
---

# Forbidden feature-folder names

When a directory is already split into **two or more** child folders that contain code, each child
must be named for **behavior or domain** — not for “leftover” file types.

❌ **Do not create junk-drawer folders:**

| Name                                       | Why                                                                         |
| ------------------------------------------ | --------------------------------------------------------------------------- |
| `helpers/`, `utils/`, `utilities/`         | Multi-export catch-alls                                                     |
| `constants/`                               | Colocate literals with the owning feature (`surface/configFilePatterns.ts`) |
| `types/`                                   | Colocate `*.type.ts` beside the feature that owns them                      |
| `common/`, `misc/`, `internal/`, `shared/` | Vague shared buckets inside a feature split                                 |

✅ **Use domain names** (from a real split):

```text
intelligenceLayer/
  workflow/     isWorkflow.ts, workflowCommands.ts
  surface/      classifyConfig.ts, configFilePatterns.ts
  detect/       detectCustomDiagnostics.ts, customDiagnosticsPatterns.ts
  prTemplate/   isPrTemplate.ts, prTemplateHasHarnessCheck.ts
```

✅ **Allowed convention folders** inside a split: `launcher/`, `config/`, `test-utils/`.

Top-level `scripts/` (and the repo root) are exempt — cross-cutting script dirs such as
`scripts/shared/` (shared helpers and `test-utils/` for script tests) may live there. The rule
applies under `packages/*/src/**`, nested `scripts/<tool>/…`, and other feature areas.

`pnpm run check:code-layout` warns on forbidden names (same `--strict` flag as flat-module
warnings).

## See also

- Code layout (authoring) index