# Comandos úteis

## Dev
- `pnpm install` — instalar dependências
- `pnpm nx run-many -t build` — build incremental
- `pnpm nx run-many -t test` — rodar todos os testes
- `pnpm nx run-many -t typecheck` — typecheck incremental
- `pnpm nx run-many -t lint` — lint de todos os projetos
- `pnpm nx affected -t test` — testes apenas dos projetos afetados
- `pnpm biome check --write .` — format + lint com auto-fix
- `pnpm biome ci .` — verificação CI (sem fix)

## Generators
- `pnpm nx g @mateusmacedo/tools:shared-lib my-lib` — criar nova shared lib

## Debug
- `pnpm nx show project <nome-do-projeto>` — config resolvida
- `pnpm nx graph` — grafo de dependências
- `pnpm nx reset` — limpar cache

## Release
- `pnpm nx release --yes` — release (normalmente via CI)
- `pnpm nx sync:check` — verificar sincronização TS references
