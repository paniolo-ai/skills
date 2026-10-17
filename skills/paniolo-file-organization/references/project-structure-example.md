---
source-slug: project-structure-example
source-hash: 948c0374112de790112b2a6877a9018d5d31ff189eaf45f0906c40b210d40ab8
bundled: 2026-10-17
title: Project Structure Example
type: concept
tags:
- authoring
- file-organization
updated: 2026-07-24
---

# Project Structure Example

### Monorepo Layout

```text
example-app/
├── api/                          # API server
│   ├── src/
│   │   ├── server.ts            # Main server & routes
│   │   ├── errors.ts            # Typed error classes
│   │   ├── schemas.ts           # Request/response schemas
│   │   ├── services.ts          # Service layer (DI)
│   │   ├── http/                # HTTP utilities (handleHttpEndpoint, errorToHttpResponse)
│   │   ├── db/                  # Database client integration
│   │   │   ├── getDbClientToken.ts
│   │   │   └── types.ts
│   │   └── user-session/        # User session management
│   │       └── getUserToken.ts
│   ├── deploy.toml             # Deployment target config
│   └── tsconfig.json
├── react/                        # React Vite frontend
│   ├── src/
│   │   ├── App.tsx              # Root component
│   │   ├── main.tsx             # Entry point
│   │   ├── components/          # Reusable components
│   │   │   ├── ItemCard.tsx
│   │   │   ├── ItemCard.test.tsx
│   │   │   └── UserMenu.tsx
│   │   ├── pages/               # Page components
│   │   │   ├── LibraryPage.tsx
│   │   │   └── NotFoundPage.tsx
│   │   ├── hooks/               # Custom hooks
│   │   │   ├── useItemLibrary.ts
│   │   │   ├── useItemLibrary.test.ts
│   │   │   └── useAuth.ts
│   │   ├── auth/                # Auth feature
│   │   │   ├── auth-slice.ts
│   │   │   ├── auth-slice.test.ts
│   │   │   └── SignInForm.tsx
│   │   ├── db/                  # Database client integration
│   │   │   ├── dbClient.ts
│   │   │   └── getDbAuthToken.ts
│   │   └── types/               # Type definitions
│   │       └── item.ts
│   └── tsconfig.json
├── shared/                       # Shared code
│   ├── src/
│   │   ├── types/
│   │   │   ├── item.ts          # Shared domain type
│   │   │   └── api.ts           # API types
│   │   ├── utils/
│   │   │   ├── formatDuration.ts
│   │   │   ├── formatDuration.test.ts
│   │   │   └── validateInput.ts
│   │   ├── schemas/             # Shared schemas
│   │   │   └── item-schema.ts
│   │   └── index.ts             # Main exports (careful: no barrel abuse)
│   └── tsconfig.json
├── docs/                         # Documentation (kebab-case filenames)
│   ├── authentication-system.md
│   ├── data-layer.md
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

---

*This skill is brought to you by [Paniolo.ai](https://paniolo.ai).*
