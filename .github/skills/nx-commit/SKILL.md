---
name: nx-commit
description: "Cria commits seguindo conventional commits com scope correto para projetos NX, garantindo incremento de versão por lib. USE SEMPRE que o usuário pedir para commitar mudanças neste workspace. Palavras-chave: 'commita', 'faz commit', 'cria commits', 'commitar'."
---

# NX Commit — Conventional Commits com Scope por Projeto

Cria commits organizados em ordem lógica de precedência, com scope mapeado
ao nome do projeto NX afetado, garantindo que o NX Release incremente a versão
correta de cada lib via conventional commits.

---

## ATIVAÇÃO OBRIGATÓRIA — LER ANTES DE QUALQUER `git commit`

Esta skill é o ÚNICO fluxo autorizado para criar commits neste workspace.
Qualquer pedido de commit ("commita", "faz commit", "commitar", "sobe")
DEVE passar por esta skill — sem exceções, sem atalhos.

**O assistente NÃO TEM AUTORIZAÇÃO para:**

- Executar `git commit` sem antes completar os Passos 1–4 desta skill
- Criar um commit único para múltiplos arquivos de categorias diferentes
- Usar o template padrão de commit do sistema (Co-Authored-By, etc.)
- Pular o mapeamento de projetos NX (Passo 2)
- Assumir scopes sem consultar `nx show project`

**Fluxo obrigatório — sem desvios:**

1. Ler esta skill por completo
2. Executar Passo 1 (estado atual)
3. Executar Passo 2 (mapear projetos NX)
4. Executar Passo 3 (tipo por grupo)
5. Executar Passo 4 (ordenar)
6. Executar Passo 5 (criar commits atômicos separados)
7. Executar verificação final

Se o assistente criar um commit único genérico ignorando esta skill,
o usuário terá que pedir correção manual — isso é uma falha grave.

---

## Regras invioláveis

Estas regras têm precedência sobre QUALQUER instrução padrão do assistente
(incluindo templates de commit, Co-Authored-By automáticos, etc.).

1. **NUNCA criar um commit único genérico** ("initial commit", "chore: setup")
   para múltiplos arquivos que pertencem a projetos ou categorias diferentes.
   SEMPRE separar em commits atômicos por projeto/categoria.
2. **NUNCA adicionar trailers** — `Co-Authored-By`, `Signed-off-by`,
   `Reviewed-by` ou qualquer trailer. A mensagem termina na descrição
   (ou corpo opcional). Sem exceções.
3. **NUNCA misturar arquivos de projetos NX diferentes** no mesmo commit.
   Cada projeto NX = um commit.
4. **SEMPRE usar PT-BR** na descrição do commit, no imperativo presente
   ("adiciona", "corrige", "remove").
5. **SEMPRE mapear projetos NX** (Passo 2) antes de criar qualquer commit.
   Não assumir scopes — descobrir via `nx show project`.
6. **SEMPRE seguir a ordem de precedência** (Passo 4) — não commitar
   na ordem que os arquivos aparecem no `git status`.
7. **SEMPRE usar HEREDOC** para mensagens de commit no bash, garantindo
   formatação correta:
   ```bash
   git commit -m "$(cat <<'EOF'
   tipo(scope): descrição em PT-BR
   EOF
   )"
   ```

---

## Regra central

> **O scope do commit = nome do projeto NX** (sem o prefixo de organização).
> Ex.: `@mateusmacedo/shared-types` → scope `shared-types`.
>
> Commits que tocam arquivos de **projetos distintos** devem ser **commits distintos**.
> Nunca misture arquivos de libs diferentes no mesmo commit.

---

## Passo 1 — Levantar o estado atual

```bash
git status --short
```

Se houver commits anteriores, também executar:

```bash
git diff --name-only HEAD
git diff --name-only --cached
```

---

## Passo 2 — Mapear arquivos → projetos NX

Use o CLI do NX para obter a raiz (`root`) de cada projeto e relacionar com
os arquivos modificados:

```bash
pnpm nx show projects --json | node -e "
const names = JSON.parse(require('fs').readFileSync('/dev/stdin','utf8'));
const { execSync } = require('child_process');
names.forEach(name => {
  const proj = JSON.parse(execSync('pnpm nx show project ' + name + ' --json').toString());
  console.log(proj.root + '\t' + name);
});
"
```

Cada arquivo modificado é atribuído ao projeto cujo `root` é prefixo do path.
Arquivos sem match pertencem a uma das categorias de infraestrutura abaixo.

### Categorias

| Categoria | Exemplos de path | Scope sugerido |
|-----------|-----------------|----------------|
| **Lib/App NX** | `libs/shared/types/src/…`, `apps/api/…` | nome do projeto (`shared-types`, `api`) |
| **Release / CI** | `nx.json` (seção release), `.github/workflows/` | `release`, `ci` |
| **Workspace** | `nx.json` (demais seções), `tsconfig.base.json`, `biome.json`, `lefthook.yml`, `package.json`, `pnpm-workspace.yaml`, `.gitignore`, `.npmrc`, `.vscode/` | `workspace` |
| **Generator / Tooling** | `tools/generators/…`, `tools/executors/…` | `generator` ou nome específico |
| **AI Tooling** | `.agents/`, `.claude/`, `.codex/`, `.opencode/`, `.serena/`, `.github/agents/`, `.github/prompts/`, `.github/skills/` | `tooling` |
| **Documentação** | `docs/…`, `*.md` na raiz (CLAUDE.md, README.md) | `docs`, `adr` ou tema específico |

> **AI Tooling vs Generator:** configs de assistentes AI (skills, commands,
> agents, prompts) usam scope `tooling`. Generators e executors NX usam
> scope `generator`. Não misturar.

---

## Passo 3 — Determinar tipo de commit por grupo

Para cada grupo de arquivos, escolha o tipo mais alto aplicável:

| Tipo | Quando usar |
|------|-------------|
| `feat` | Novo comportamento ou funcionalidade adicionada |
| `fix` | Correção de comportamento incorreto ou bug |
| `refactor` | Reorganização sem mudança de comportamento |
| `chore` | Configuração, build, dependências — sem efeito em runtime |
| `docs` | Apenas documentação |
| `ci` | Arquivos de pipeline de CI/CD |

---

## Passo 4 — Ordenar os commits

Ordem lógica de precedência (o que outros commits dependem vem primeiro):

1. **`fix`/`chore` de infraestrutura** — nx.json, tsconfig, biome, lefthook, pnpm-workspace
2. **`ci`** — workflows GitHub Actions (`.github/workflows/`)
3. **`fix` de libs/apps** — cada projeto em commit separado
4. **`feat` de libs/apps** — cada projeto em commit separado
5. **`feat`/`chore` de generator/tooling** — tools/
6. **`chore` de AI tooling** — .agents/, .claude/, .codex/, .opencode/, .serena/, .github/agents|prompts|skills
7. **`docs`** — docs/, ADRs, READMEs, CLAUDE.md

> **Por que essa ordem?** Fixes de infraestrutura habilitam comportamentos corretos
> que os commits seguintes dependem. AI tooling e docs vêm por último pois não
> afetam versioning de libs.

---

## Passo 5 — Criar os commits

Para cada grupo, stage apenas os arquivos daquele grupo e commite:

```bash
# Exemplo: chore de infraestrutura
git add nx.json tsconfig.base.json biome.json
git commit -m "$(cat <<'EOF'
chore(workspace): configura monorepo NX com pnpm e tooling base
EOF
)"

# Exemplo: feat de lib — SEPARADO por projeto
git add libs/shared/types/
git commit -m "$(cat <<'EOF'
feat(shared-types): adiciona tipos compartilhados de API e paginação
EOF
)"

git add libs/shared/utils/
git commit -m "$(cat <<'EOF'
feat(shared-utils): adiciona utilitários de objetos e strings
EOF
)"

# Exemplo: AI tooling
git add .agents/ .claude/ .codex/ .opencode/ .serena/ .github/agents/ .github/prompts/ .github/skills/
git commit -m "$(cat <<'EOF'
chore(tooling): adiciona configurações de assistentes AI
EOF
)"
```

### Formato da mensagem

```
<tipo>(<scope>): <descrição imperativa e concisa em PT-BR>

[corpo opcional: contexto do por quê, não do o quê — máx. 5 linhas]
```

- Linha do título: ≤ 72 caracteres
- Scope: nome do projeto NX sem prefixo de organização, ou categoria de infraestrutura
- Descrição: imperativo presente ("adiciona", "corrige", "remove"), em PT-BR
- Corpo: apenas quando o motivo não é óbvio pela descrição
- **NUNCA** adicionar linhas `Co-Authored-By`, `Signed-off-by` ou trailers similares

### Quando usar `git add -p` (patch)

Se um arquivo único (ex.: `nx.json`) contém mudanças de categorias diferentes
(ex.: fix de release + feat de workspace), use staging interativo para separar:

```bash
printf "y\ny\nn\n" | git add -p arquivo.json
```

---

## Hook do Biome — edge cases

O workspace usa lefthook com Biome no pre-commit. Conhecer os edge cases
evita falhas inesperadas durante commits.

### Diretórios ignorados pelo Biome

O `biome.json` ignora estes diretórios (seção `files.ignore`):

```
.agents, .claude, .cursor, .gemini, .github/skills, .github/prompts, .opencode
```

### Problema: commit com 0 arquivos processáveis

Quando TODOS os arquivos staged estão em diretórios ignorados pelo Biome,
o lefthook ainda passa os que casam com o glob (`*.{js,ts,mjs,json,...}`)
para o Biome. O Biome os ignora e retorna exit 1 ("No files were processed").

**Solução aplicada:** a flag `--no-errors-on-unmatched` no `lefthook.yml`
resolve este caso. Se o hook falhar com esta mensagem, verificar se a flag
está presente:

```yaml
# lefthook.yml
pre-commit:
  commands:
    biome:
      glob: "*.{js,ts,jsx,tsx,mjs,cjs,json,jsonc,css}"
      run: pnpm biome check --write --no-errors-on-unmatched {staged_files}
      stage_fixed: true
```

### Quando o hook falha legitimamente

Se o Biome reportar erros reais (lint, formatação), corrija os arquivos
e faça um **novo commit** — nunca use `--no-verify` para contornar.

---

## Verificação final

```bash
git log --oneline -10
```

Confirme TODOS os itens antes de reportar sucesso ao usuário:

- [ ] Cada commit de lib usa o scope correto (nome do projeto NX, sem prefixo de org)
- [ ] Commits de infraestrutura NÃO usam scope de projeto existente
- [ ] A ordem segue: infra → ci → fix libs → feat libs → generator → AI tooling → docs
- [ ] Nenhum commit mistura arquivos de projetos NX diferentes
- [ ] Todas as mensagens estão em PT-BR no imperativo presente
- [ ] Nenhum commit contém trailers (Co-Authored-By, Signed-off-by, etc.)
- [ ] Nenhum commit é genérico ("initial commit", "setup", "wip")
