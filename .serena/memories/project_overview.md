# nx-template — @mateusmacedo

Monorepo Nx multistack (Express, Fastify, NestJS, Next.js, Angular) da org @mateusmacedo.

## Tech Stack
- **Runtime**: Node.js 22+, TypeScript 5.9, ES2022, nodenext modules
- **Package manager**: pnpm 9+ (workspace protocol)
- **Build**: Nx 22.6 com TypeScript Project References + SWC
- **Lint/Format**: Biome (single quotes, 2 spaces, 100 line width, semicolons always)
- **Test**: Jest 30 via @swc/jest
- **CI**: GitHub Actions (ubuntu-latest), nx affected, cache pnpm + Nx
- **Release**: Nx Release, independent versioning, conventional commits, GitHub Packages
- **Git hooks**: Lefthook (pre-commit: biome check, pre-push: lint/typecheck/test/build)
- **Go**: go.work para pacotes Go via @nx-go/nx-go

## Structure
```
apps/           → aplicações (tag:type:app)
libs/shared/    → libs agnósticas (scope:shared)
libs/backend/   → libs de servidor (scope:backend)
libs/frontend/  → libs de UI (scope:frontend)
libs/data-access/ → clients HTTP
tools/generators/ → Nx generators (@mateusmacedo/tools)
packages/       → Go packages
```

## Naming
- NPM scope: `@mateusmacedo`
- Libs: `@mateusmacedo/shared-{name}`, `@mateusmacedo/backend-{name}`, etc.
- Tags obrigatórias: `type:`, `scope:`, `stack:`

## Branch model
- Default branch: `main`
- Release via PR merge de `release/*` ou `hotfix/*` em `main`
