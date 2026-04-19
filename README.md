# nx-template

Baseline de monorepo Nx multistack — Express, Fastify, NestJS, Next.js e Angular no mesmo workspace.

## Stack suportada

| Stack                     | Tipo        | Plugin Nx                     |
| ------------------------- | ----------- | ----------------------------- |
| Node.js / TypeScript puro | libs        | `@nx/js`                      |
| Express                   | apps        | `@nx/express` + `@nx/webpack` |
| Fastify                   | apps        | `@nx/node` + `@nx/webpack`    |
| NestJS                    | apps + libs | `@nx/nest` + `@nx/webpack`    |
| Next.js                   | apps + libs | `@nx/next`                    |
| Angular                   | apps + libs | `@nx/angular`                 |

## Estrutura de diretórios

```
nx-template/
├── apps/                        # Aplicações executáveis
│   └── <stack>-<name>/
│   └── <stack>-<name>-e2e/
├── libs/
│   ├── shared/                  # Libs agnósticas de framework (scope:shared)
│   │   ├── utils/               # Utilitários TypeScript puros
│   │   ├── types/               # Tipos e interfaces compartilhados
│   │   └── testing/             # Fixtures e helpers de teste
│   ├── backend/                 # Libs de servidor (scope:backend)
│   │   ├── domain/              # Lógica de domínio
│   │   ├── nest/                # Módulos NestJS reutilizáveis
│   │   └── infra/               # Adapters (DB, cache, queue)
│   ├── frontend/                # Libs de UI (scope:frontend)
│   │   ├── ui/                  # Componentes React/Next.js
│   │   └── angular/             # Componentes Angular
│   └── data-access/
│       └── api-client/          # Client HTTP compartilhado
├── tools/
│   ├── generators/              # Nx generators da organização
│   │   └── shared-lib/          # nx g @mateusmacedo/tools:shared-lib
│   └── executors/               # Nx executors customizados
├── docs/
│   └── adr/                     # Architecture Decision Records
├── .github/workflows/           # CI, release e create-release
├── nx.json
├── tsconfig.base.json           # TypeScript Project References (nodenext)
├── pnpm-workspace.yaml          # apps/*, libs/**, tools/*
├── biome.json                   # Biome (lint + format)
├── jest.config.ts               # Jest root
├── jest.preset.js               # Preset SWC
└── .spec.swcrc                  # Config SWC para testes (suporta decorators)
```

## Convenção de tags (obrigatória em todo project.json)

Cada projeto deve declarar uma tag de cada dimensão:

```json
{
  "tags": ["type:app", "scope:backend", "stack:nest"]
}
```

| Dimensão | Valores                                                 |
| -------- | ------------------------------------------------------- |
| `type:`  | `app`, `lib`, `e2e`                                     |
| `scope:` | `shared`, `backend`, `frontend`                         |
| `stack:` | `node`, `express`, `fastify`, `nest`, `next`, `angular` |

## Comandos principais

```bash
# Rodar lint apenas nos projetos afetados pelo último commit
pnpm nx affected -t lint

# Typecheck incremental de tudo
pnpm nx run-many -t typecheck

# Build incremental de tudo
pnpm nx run-many -t build

# Ver grafo de dependências
pnpm nx graph

# Gerar nova shared lib
pnpm nx g @mateusmacedo/tools:shared-lib my-lib
```

## Adicionar uma nova app

```bash
# Express
pnpm nx g @nx/express:app my-api --directory=apps/my-api

# NestJS
pnpm nx g @nx/nest:app my-nest-api --directory=apps/my-nest-api

# Next.js
pnpm nx g @nx/next:app my-web --directory=apps/my-web

# Angular
pnpm nx g @nx/angular:app my-angular --directory=apps/my-angular
```

Sempre adicione as tags obrigatórias no `project.json` gerado.

## CI/CD

### CI (`.github/workflows/ci.yml`)

Roda em PRs para `main`, `develop` e `release/**`. Usa `ubuntu-latest` com
cache de pnpm store e Nx. Executa sobre os projetos **afetados** (`nx affected`):

1. `biome ci` — formatação e linting (Biome)
2. `lint` — regras adicionais por projeto
3. `typecheck` — TypeScript Project References
4. `test` — Jest + SWC (com coverage)
5. `build` — build incremental
6. `e2e` — testes E2E (parallelism=1)

### Release (`.github/workflows/release.yml`)

Dispara ao mergear PRs de branches `release/*` ou `hotfix/*` em `main`,
ou via workflow dispatch manual. Usa `pnpm nx release --yes` com versionamento
independente e conventional commits. Publica pacotes no GitHub Packages
(`@mateusmacedo` scope) e, se houver apps com tag `type:app`, publica imagens
Docker no GitHub Container Registry (ghcr.io).

### Create Release (`.github/workflows/create-release.yml`)

Workflow dispatch manual que detecta o tipo de bump (major/minor/patch) a
partir dos commits convencionais entre `main` e `develop`, cria a branch
`release/X.Y.Z` e abre um PR com checklist de validação.

## Decisões arquiteturais

Consulte `docs/adr/` para o registro de decisões.

- **ADR-001** — Baseline do monorepo multistack (preset, módulos, tags, SWC)
- **ADR-002** — Configuração de tasks, cache e pipelines no Nx
