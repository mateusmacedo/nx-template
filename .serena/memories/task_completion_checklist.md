# O que fazer ao concluir uma task

1. `pnpm biome check --write .` — format + lint auto-fix
2. `pnpm nx affected -t lint` — verificar lint
3. `pnpm nx affected -t typecheck` — verificar tipos
4. `pnpm nx affected -t test` — rodar testes afetados
5. `pnpm nx affected -t build` — verificar build

Lefthook roda automaticamente no pre-push, mas verificar antes manualmente evita surpresas.
