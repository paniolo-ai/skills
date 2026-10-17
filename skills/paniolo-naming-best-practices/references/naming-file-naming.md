---
source-slug: naming-file-naming
source-hash: 353cb650c60d96305b6f6a8aaae5b8ce3508e9ed00e7c72b97ad2f544ff4524c
bundled: 2026-10-17
title: File Naming
type: concept
tags:
- authoring
- naming
- typescript
updated: 2026-06-18
---

# File Naming

| What                             | Convention                         | Example                                             |
| -------------------------------- | ---------------------------------- | --------------------------------------------------- |
| React component                  | PascalCase `.tsx`                  | `EventManageView.tsx`                               |
| Single default-exported function | camelCase `.ts` (never kebab-case) | `computeEventPermissions.ts`, `fetchEventBySlug.ts` |
| Multi-symbol file                | kebab-case `.ts`                   | `auth-utils.ts`                                     |
| Single type export (types only)  | PascalCase `.type.ts`              | `ShareCreateRequest.type.ts`                        |
| Multiple types only (exports)    | kebab-case `.type.ts`              | `user-live-chain-setters.type.ts`                   |
| Test file                        | same name + `.test.ts/.tsx`        | `computeEventPermissions.test.ts`                   |
| Test helper                      | same name + `.test-util.ts`        | `makeUseManageView.test-util.ts`                    |
| Directory                        | kebab-case                         | `event-manage-view/`, `community-search-input/`     |
| Doc files                        | kebab-case `.md`                   | `authentication-system.md`                          |

For complete file organization rules see `/skills/file-organization/SKILL.md`.

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
