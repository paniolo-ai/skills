---
source-slug: naming-file-naming
source-hash: d043da6a2534684c43835b5c383de053bebe771c47b434925208380c3e7c85d7
bundled: 2026-10-16
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
