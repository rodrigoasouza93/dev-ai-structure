# Instruções para Agentes — Bliss Task Execution

Este é o projeto agregador de atividades que envolvem múltiplos projetos Bliss.
Os projetos ficam no diretório de trabalho local (ex.: `~/Documents/bliss/`).

## Primeiro passo

- Leia o `SKILL.md` em `.claude/skills/<nome>/` antes de implementar ou revisar.
- Crie ou atualize uma spec em `tasks/prd-[nome]/` antes de trabalhos amplos ou que alterem contratos.
- Escreva specs e PRDs em Português (pt-BR). Mantenha identificadores de código, campos de API, caminhos e nomes externos no idioma original.
- Prefira contexto local do repositório antes de suposições. Use `rg` e testes/exemplos existentes antes de criar novos padrões.

## Stack dos projetos Bliss

- **Linguagem**: TypeScript (estritamente, sem JavaScript)
- **Package manager**: `pnpm` (nunca npm ou yarn)
- **Runtime**: Node.js 20+
- **Deploy**: AWS Lambda via Serverless Framework
- **HTTP**: Fastify (maioria), NestJS (bliss-broker-gateway)
- **Database**: PostgreSQL via Prisma ORM
- **Testes**: Vitest
- **Frontend** (intranet): React + Vite + Ant Design + TanStack Query

## Comandos comuns

- Instalar dependências: `pnpm install`
- Verificar tipos: `pnpm type-check` ou `pnpm run type-check`
- Lint: `pnpm lint`
- Testes: `pnpm test`
- Deploy staging: `pnpm run deploy:staging`
- Deploy production: `pnpm run deploy:production`

Execute a menor verificação útil primeiro; amplie quando a mudança tocar contratos ou infraestrutura compartilhada.

## Convenções de Git

- **Sempre crie uma branch nova antes de implementar uma task** (`execute-task`). Nunca implemente ou commite direto em `main`/`master`. Antes de criar a branch:
  1. `git checkout main` (ou `master`) e `git pull --ff-only` para sincronizar com o remoto.
  2. Confirme que não há uma branch de feature antiga desviada (`git branch --show-current`) — se o repositório já estiver em outra branch local, troque para `main` primeiro em vez de criar a nova branch em cima dela.
  3. Crie a branch a partir do `main` já atualizado: `git checkout -b <nome-da-branch>`.
  4. Antes de commitar, confirme que a branch parte do `main` remoto sem divergência: `git merge-base HEAD origin/main` deve ser igual a `git rev-parse origin/main`.
- **Nomes de branch**: sempre no padrão Conventional Commits — `tipo/imp-NNN-slug-curto-em-kebab-case` (ex: `feat/imp-409-comprovante-endereco-empresa`). `tipo` é `feat`, `fix`, `chore`, `refactor`, `test` ou `docs`, conforme a natureza da mudança.
- **Nunca** prefixar o nome da branch com o usuário (ex: `rodrigosouza/...`) — inclusive ignore o `gitBranchName` sugerido automaticamente pelo Linear, que usa esse formato.
- **Mensagens de commit**: também no padrão Conventional Commits (`tipo(escopo): descrição`).

## Fluxo de desenvolvimento

Para trabalho de feature, siga esta sequência com gates de aprovação. Cada
etapa é uma skill em `.claude/skills/` (invoque por `/nome` ou deixe o modelo
acioná-la pelo contexto):

```
PRD (skill create-prd)
  → aprovação do usuário
    → TechSpec (skill create-techspec)
      → aprovação do usuário
        → Tasks (skill create-tasks)
          → aprovação do usuário
            → Implementação (skill execute-task) + @task-reviewer
              → QA (skill execute-qa)
                → Bugfix se necessário (skill execute-bugfix)
                  → Code Review (skill execute-review)
```

**Gate de aprovação:** cada documento deve conter `Status: APROVADO PELO USUÁRIO` antes da próxima fase. Nunca avance sem esta confirmação.

## Estrutura de tasks

Armazene todo o trabalho de feature em `./tasks/prd-[nome-em-kebab-case]/`:

```
tasks/prd-[nome]/
├── prd.md
├── techspec.md
├── tasks.md
├── 1_task.md
├── 2_task.md
├── qa.md
├── bugs.md          (somente se bugs foram encontrados)
├── bugfix.md        (somente se bugs foram corrigidos)
└── codereview.md
```

## Skills

Use a skill correspondente quando o trabalho segue um fluxo repetível.

### Skills do fluxo de feature (PRD → TechSpec → Tasks → Implementação → QA → Bugfix → Review)

| Skill | Acionar para… | Não usar se… |
|-------|--------------|--------------|
| `create-prd` | Iniciar feature, capturar requisitos, gerar `prd.md` | Já existe PRD aprovado ou é só ajuste técnico |
| `create-techspec` | Desenhar arquitetura/decisões técnicas a partir do PRD aprovado | `prd.md` não está `APROVADO PELO USUÁRIO` |
| `create-tasks` | Quebrar a feature em tarefas a partir de PRD + TechSpec aprovados | PRD ou TechSpec não aprovados |
| `execute-task` | Implementar a próxima tarefa de `tasks.md` aprovado | `tasks.md` não aprovado |
| `execute-qa` | Validar a implementação, rodar testes e a11y, gerar `qa.md` | Ainda não há implementação |
| `execute-bugfix` | Corrigir os bugs de `bugs.md` com testes de regressão | Não há `bugs.md` com bugs pendentes |
| `execute-review` | Code review final via `git diff`, gerar `codereview.md` | Nada implementado para revisar |

### Skills de apoio (padrões e tecnologias)

| Skill | Acionar para… | Não usar se… |
|-------|--------------|--------------|
| `code-standards-en` | Nomes em inglês, CQS, early return, tamanho de métodos/classes | Política exige identificadores localizados |
| `context7` | Documentação técnica atualizada de bibliotecas, frameworks, SDKs, APIs | A resposta puder ser dada só pelo contexto local sem risco de versão |
| `nodejs-typescript-conventions` | TS/Node, ESM, pnpm, async/await, sem `any` | Projeto JS puro |
| `vitest-testing` | Vitest, `vi`, AAA, timers, integração HTTP sem supertest | Jest/Sinon como stack principal de mock |
| `fastify-rest-http` | Rotas Fastify, plugins, schemas, status codes, HTTP externo | Framework HTTP não for Fastify |
| `serverless-aws-lambda` | Lambda handlers, Serverless Framework config, stages, event sources | Deploy não for AWS Lambda |
| `prisma-orm` | Modelos Prisma, migrations, queries, transactions, connection pooling | ORM diferente de Prisma |
| `react-frontend-conventions` | React FC, TSX, Ant Design, hooks, testes de UI (intranet) | Class components, projeto não for intranet |
| `repo-folder-structure` | Onde criar features, pages, controllers/services/data, Lambda functions | Monorepo ou framework diferente do template |
| `ui-ux-pro-max` | Design/revisão de UI (componentes, páginas, paletas, tipografia, a11y) | Tarefa só backend/API sem interface |
| `linear` | Ler, criar ou atualizar issues, projetos, labels e cycles no Linear via MCP | MCP do Linear não estiver configurado |

**Ordem sugerida por tarefa:**
- **Lambda Backend**: `serverless-aws-lambda` → `fastify-rest-http` (se Fastify) → `nodejs-typescript-conventions` → `code-standards-en`
- **Backend com Prisma**: acrescentar `prisma-orm`
- **Frontend (intranet)**: `ui-ux-pro-max` → `react-frontend-conventions` → `repo-folder-structure` → `nodejs-typescript-conventions` → `code-standards-en`
- **Documentação técnica externa**: `context7` antes de implementar
- **Testes**: `vitest-testing` + skill da camada testada
- **Gestão de tickets**: `linear` para criar/atualizar issues, planejar sprints ou auditar workload

## Persistência do Modo Plano

<plan_file>`.codex/plans/[timestamp]-[plan-slug].md`</plan_file>

- **OBRIGATÓRIO ABSOLUTO**: No modo Plano, após o usuário aceitar um plano, **SEMPRE** escreva o plano aceito em um arquivo Markdown dentro de <plan_file>.
- **OBRIGATÓRIO**: Se o plano aceito for atualizado posteriormente, atualize ou adicione o respectivo arquivo Markdown em <plan_file>.

## Context7

Use Context7 para documentação atual antes de alterar código que depende de bibliotecas ou APIs externas. Ver `.claude/skills/context7/SKILL.md`.

## Áreas de alto risco

Solicite confirmação explícita do usuário antes de alterar:

- Schemas de API públicas, payloads de resposta ou comportamento de status codes
- Configuração de deploy Serverless (domínios, IAM, layers, runtime, packaging)
- Dependências, lockfiles ou configurações do package manager
- Variáveis de ambiente, secrets ou logging de dados sensíveis
- Migrations de banco de dados em produção
