---
source-wiki: sharp-shooter-wiki
source-slug: naming-file-naming
source-hash: 9ff07b45eaa925f7e5cef9420b65e700465a33c7796f58fe06826c7d5f96d543
bundled: 2026-08-01
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

For complete file organization rules see
`/skills/file-organization/SKILL.md`.

## See also

- Naming conventions (authoring) index
