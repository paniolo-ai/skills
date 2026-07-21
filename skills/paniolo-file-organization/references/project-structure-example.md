---
source-wiki: sharp-shooter-wiki
source-slug: project-structure-example
source-hash: d64637812821636e8278731bc8e415417a389f05e53b721f0c4672e42f851c2c
bundled: 2026-07-20
title: Project Structure Example
type: concept
tags:
- authoring
- file-organization
updated: 2026-06-18
sources:
- raw/harness-eng/songshare-ai-system/skill-file-organization.md
---

# Project Structure Example

### Monorepo Layout

```text
songshare-effect/
├── api/                          # Hono API server (Effect-TS)
│   ├── src/
│   │   ├── server.ts            # Main server & routes
│   │   ├── errors.ts            # Typed error classes
│   │   ├── schemas.ts           # Effect schemas
│   │   ├── services.ts          # Service layer (DI)
│   │   ├── http/                # HTTP utilities (handleHttpEndpoint, errorToHttpResponse)
│   │   ├── supabase/            # Supabase integration
│   │   │   ├── getSupabaseClientToken.ts
│   │   │   └── types.ts
│   │   └── user-session/        # User session management
│   │       └── getUserToken.ts
│   ├── wrangler.toml            # Cloudflare Workers config
│   └── tsconfig.json
├── react/                        # React Vite frontend
│   ├── src/
│   │   ├── App.tsx              # Root component
│   │   ├── main.tsx             # Entry point
│   │   ├── components/          # Reusable components
│   │   │   ├── SongCard.tsx
│   │   │   ├── SongCard.test.tsx
│   │   │   └── UserMenu.tsx
│   │   ├── pages/               # Page components
│   │   │   ├── LibraryPage.tsx
│   │   │   └── NotFoundPage.tsx
│   │   ├── hooks/               # Custom hooks
│   │   │   ├── useSongLibrary.ts
│   │   │   ├── useSongLibrary.test.ts
│   │   │   └── useAuth.ts
│   │   ├── auth/                # Auth feature
│   │   │   ├── auth-slice.ts
│   │   │   ├── auth-slice.test.ts
│   │   │   └── SignInForm.tsx
│   │   ├── supabase/            # Supabase integration
│   │   │   ├── supabaseClient.ts
│   │   │   └── getSupabaseAuthToken.ts
│   │   └── types/               # Type definitions
│   │       └── song.ts
│   └── tsconfig.json
├── shared/                       # Shared code
│   ├── src/
│   │   ├── types/
│   │   │   ├── song.ts          # Shared song type
│   │   │   └── api.ts           # API types
│   │   ├── utils/
│   │   │   ├── formatDuration.ts
│   │   │   ├── formatDuration.test.ts
│   │   │   └── validateInput.ts
│   │   ├── schemas/             # Shared schemas
│   │   │   └── song-schema.ts
│   │   └── index.ts             # Main exports (careful: no barrel abuse)
│   └── tsconfig.json
├── docs/                         # Documentation (kebab-case filenames)
│   ├── authentication-system.md
│   ├── effect-implementation.md
│   └── file-organization.md
├── scripts/                      # Build & utility scripts
│   ├── build/
│   │   └── generate-schemas/
│   │       ├── index.ts
│   │       └── helpers/
│   │           └── toPascalCase.ts
│   └── dev/
├── .github/
│   ├── skills/                   # Agent Skills
│   │   └── react-best-practices/
│   │       ├── SKILL.md
│   │       └── references/
│   │           └── REFERENCE.md
│   └── agents/                   # Custom agents
│       └── Comment Agent.agent.md
├── vite.config.ts               # ESM
├── tailwind.config.js           # ESM
└── tsconfig.json
```

## See also

- File organization (authoring) index