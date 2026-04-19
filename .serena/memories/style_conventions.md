# Convenções de código e estilo

## TypeScript
- target: ES2022, module: nodenext, strict: true
- Single quotes, semicolons always, trailing commas (ES5)
- 2-space indent, 100 char line width
- `type` over `interface` (useImportType: error)
- No unused variables/imports (error)
- No explicit any (warn)
- Max cognitive complexity: 30 (warn)

## Commits
- Conventional Commits: `type(scope): descrição em PT-BR`
- Scope = nome do projeto NX sem prefixo de org
- Nunca misturar arquivos de projetos distintos no mesmo commit
- Ordem: infra → ci → fix libs → feat libs → tooling → docs
- NUNCA adicionar trailers (Co-Authored-By, Signed-off-by, etc.)
- Seguir skill nx-commit (.agents/skills/nx-commit/SKILL.md) para todo commit

## Tags (obrigatórias em project.json)
- `type:lib` | `type:app` | `type:e2e`
- `scope:shared` | `scope:backend` | `scope:frontend`
- `stack:node` | `stack:react` | `stack:angular` | `stack:universal`

## Publicação
- Registry: GitHub Packages (@mateusmacedo scope)
- Versioning: independent por lib
- Release tag: `{projectName}@{version}`
