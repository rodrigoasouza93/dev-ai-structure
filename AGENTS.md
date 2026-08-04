# Instruções para Agentes — Bliss Task Execution

Este é o projeto agregador de atividades que envolvem múltiplos projetos Bliss.
Os projetos ficam no diretório de trabalho local (ex.: `~/Documents/bliss/`).

## Primeiro passo

- Leia integralmente o `SKILL.md` aplicável em `.agents/skills/<nome>/` antes de implementar ou revisar. Esse é o catálogo canônico, compartilhado por Claude Code, Cursor e Codex/ChatGPT.
- Crie ou atualize uma spec em `tasks/prd-[nome]/` antes de trabalhos amplos ou que alterem contratos.
- Escreva specs e PRDs em Português (pt-BR). Mantenha identificadores de código, campos de API, caminhos e nomes externos no idioma original.
- Prefira contexto local do repositório antes de suposições. Use `rg` e testes/exemplos existentes antes de criar novos padrões.

## Stack dos projetos Bliss

- **Linguagem**: TypeScript (estritamente, sem JavaScript)
- **Package manager**: `pnpm` (nunca npm ou yarn)
- **Runtime**: Node.js 20+
- **Deploy**: AWS Lambda, com IaC em Serverless Framework **ou** AWS CDK (ver "Infraestrutura e deploy")
- **HTTP**: Fastify (maioria), NestJS (bliss-broker-gateway)
- **Database**: PostgreSQL via Prisma ORM
- **Testes**: Vitest
- **Frontend** (intranet): React + Vite + Ant Design + TanStack Query

## Comandos comuns

- Instalar dependências: `pnpm install`
- Verificar tipos: `pnpm type-check` ou `pnpm run type-check`
- Lint: `pnpm lint`
- Testes: `pnpm test`
- Deploy (Serverless Framework): `pnpm run deploy:staging` / `pnpm run deploy:production`
- Deploy (AWS CDK): `pnpm exec cdk diff -c environment=dev` / `pnpm exec cdk deploy -c environment=<dev|prod>`, executados dentro de `bliss-cdk/`

Execute a menor verificação útil primeiro; amplie quando a mudança tocar contratos ou infraestrutura compartilhada.

## Infraestrutura e deploy

Os serviços Bliss rodam em AWS Lambda, mas há **duas pilhas de IaC em uso**, e as duas são válidas. Descubra qual o repositório usa antes de mexer em infra: `serverless.ts` na raiz indica Serverless Framework; uma pasta `bliss-cdk/` com `catalog/` indica CDK.

| | Serverless Framework | AWS CDK |
|---|---|---|
| Declaração | `serverless.ts` (`@serverless/typescript`) + plugins (`serverless-esbuild`, `serverless-domain-manager`, `serverless-iam-roles-per-function`) | `bliss-cdk/catalog/functions/*.yaml` + `catalog/http-api.yaml`, lidos por `@saude-bliss/bliss-aws-cdk-modules` |
| Stages | `sandbox` / `production` | `dev` / `prod` (via `-c environment=`) |
| Deploy | `pnpm run deploy:staging` / `deploy:production` | GitHub Actions com OIDC: `cdk diff` em PR, `cdk deploy` em push na `main` (dev) e em tag `v*` (prod) |
| API Gateway | REST API v1 (`http`) por padrão | HTTP API v2 |
| Referências | `bliss-insurer-edge`, `bliss-order-gateway`, `bliss-validation-job` | `bliss-auth-gateway` |

**Como escolher:**

- **Serviço existente**: use a pilha que ele já tem. Migrar de IaC é mudança própria, com PRD/TechSpec dedicados — nunca carona em uma feature.
- **Serviço novo**: prefira **CDK**, que é o padrão mais recente e dispensa a cadeia de plugins do Serverless. Escolha Serverless Framework quando o serviço precisar de algo que só ele entrega bem no nosso contexto (ex.: `serverless-offline` como parte do fluxo local, ou paridade obrigatória com um serviço irmão que já é Serverless).
- **Registre a escolha na TechSpec**, em "Principais decisões", com a justificativa — inclusive quando ela seguir o padrão.

**Armadilhas conhecidas:**

- Os nomes de ambiente **não coincidem** entre as pilhas (`sandbox`/`production` vs `dev`/`prod`). Ao integrar dois serviços, confirme a que ambiente cada URL corresponde em vez de inferir pelo nome.
- No CDK, a infra vive isolada em `bliss-cdk/` com `package.json` e lockfile próprios (pnpm), separada do app. `@saude-bliss/bliss-aws-cdk-modules` é pacote restricted: o install precisa de `NPM_READ_TOKEN`.
- `cdk synth`/`diff` não exigem credenciais AWS válidas; `deploy`/`destroy` exigem sessão ativa (`aws sso login --profile <perfil>`).
- Deploy — em qualquer das duas pilhas — **não** aplica migration nem seed. Verifique o banco direto antes de afirmar que uma feature está ativa em um ambiente.

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
etapa é uma skill em `.agents/skills/` (invoque por `/nome` quando o cliente
oferecer esse comando ou deixe o modelo
acioná-la pelo contexto):

```
Ideação (skill create-ideation)   [opcional — quando a ideia/direção ainda não está definida]
  → aprovação do usuário
    → Plano de Execução (skill create-execution-plan)   [opcional — quando a ideia se desdobra em várias frentes/PRDs]
      → aprovação do plano
    → PRD (skill create-prd)
      → aprovação do usuário
        → TechSpec (skill create-techspec)
          → aprovação do usuário
            → Tasks (skill create-tasks)
              → aprovação do usuário
                → Implementação (skill execute-task por task ou execute-feature em loop) + agente task-reviewer
                  → QA (skill execute-qa)
                    → Bugfix se necessário (skill execute-bugfix)
                      → Code Review (skill execute-review)
```

**Gate de aprovação:** cada documento deve conter `Status: APROVADO PELO USUÁRIO` antes da próxima fase. Nunca avance sem esta confirmação.

## Estrutura de ideação

Explorações de ideias (fase anterior ao PRD) ficam em `./ideas/[nome-em-kebab-case]/`:

```
ideas/[nome]/
├── idea.md
└── plano-execucao.md   (opcional — quando a ideia se desdobra em vários PRDs)
```

Uma vez aprovada (`Status: APROVADO PELO USUÁRIO`), a ideia vira insumo para a skill `create-prd`. Quando a direção recomendada se desdobra em várias frentes, use antes a skill `create-execution-plan` para decompô-la em pontos de execução (`plano-execucao.md`) e gerar um PRD por ponto. O `plano-execucao.md` canônico mora aqui em `ideas/[nome]/`; a pasta da iniciativa em `tasks/` mantém apenas um ponteiro para ele (ver abaixo).

## Estrutura de tasks

**Feature avulsa** (sem plano de execução) — armazene em `./tasks/prd-[nome-em-kebab-case]/`:

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

**Iniciativa com plano de execução** (vários PRDs de uma mesma ideia) — agrupe tudo sob uma pasta da iniciativa, com uma subpasta por ponto de execução (cada subpasta é um workspace de feature auto-contido, com a mesma estrutura acima):

```
tasks/[nome-da-iniciativa]/          (mesmo slug da ideia em ideas/)
├── plano-execucao.md                (ponteiro para ideas/[nome]/plano-execucao.md)
├── pe-1-[slug-do-ponto]/
│   ├── prd.md
│   ├── techspec.md
│   ├── tasks.md
│   ├── 1_task.md
│   ├── qa.md
│   └── codereview.md
├── pe-2-[slug-do-ponto]/
│   └── prd.md ...
└── pe-3-[slug-do-ponto]/
    └── prd.md ...
```

## Skills

Use a skill correspondente quando o trabalho segue um fluxo repetível.

### Skills do fluxo de feature (Ideação → PRD → TechSpec → Tasks → Implementação → QA → Bugfix → Review)

| Skill | Acionar para… | Não usar se… |
|-------|--------------|--------------|
| `create-ideation` | Explorar ideias/soluções/projetos possíveis antes do PRD; divergir, comparar trade-offs e recomendar; gerar `idea.md` | O problema já está definido e você só precisa de requisitos (use `create-prd`) |
| `create-execution-plan` | Decompor uma ideação aprovada e ampla em pontos de execução (`plano-execucao.md`) e gerar um PRD por ponto via `create-prd` | A ideia recomendada é uma única frente (chame `create-prd` direto) ou a ideação não está aprovada |
| `create-prd` | Iniciar feature, capturar requisitos, gerar `prd.md` | Já existe PRD aprovado ou é só ajuste técnico |
| `create-techspec` | Desenhar arquitetura/decisões técnicas a partir do PRD aprovado | `prd.md` não está `APROVADO PELO USUÁRIO` |
| `create-tasks` | Quebrar a feature em tarefas a partir de PRD + TechSpec aprovados | PRD ou TechSpec não aprovados |
| `execute-feature` | Executar em sequência todas as tasks aprovadas, com review, autocorreção e commit local por task | `tasks.md` não aprovado ou o usuário quiser executar apenas uma task |
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
| `serverless-aws-lambda` | Lambda handlers, event sources, IAM, cold start — e a config em `serverless.ts` quando a pilha for Serverless Framework | Deploy não for AWS Lambda |
| `prisma-orm` | Modelos Prisma, migrations, queries, transactions, connection pooling | ORM diferente de Prisma |
| `react-frontend-conventions` | React FC, TSX, Ant Design, hooks, testes de UI (intranet) | Class components, projeto não for intranet |
| `repo-folder-structure` | Onde criar features, pages, controllers/services/data, Lambda functions | Monorepo ou framework diferente do template |
| `ui-ux-pro-max` | Design/revisão de UI (componentes, páginas, paletas, tipografia, a11y) | Tarefa só backend/API sem interface |
| `linear` | Ler, criar ou atualizar issues, projetos, labels e cycles no Linear via MCP | MCP do Linear não estiver configurado |

**Ordem sugerida por tarefa:**
- **Lambda Backend**: `serverless-aws-lambda` → `fastify-rest-http` (se Fastify) → `nodejs-typescript-conventions` → `code-standards-en`. A parte de handler/event source/IAM da skill vale nas duas pilhas de IaC; a de `serverless.ts` só na pilha Serverless Framework. Para CDK, use `bliss-auth-gateway` como referência de catálogo (não há skill própria ainda).
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

Use Context7 para documentação atual antes de alterar código que depende de bibliotecas ou APIs externas. Ver `.agents/skills/context7/SKILL.md`.

## Compatibilidade entre clientes de IA

- **Fonte comum de instruções:** `AGENTS.md`.
- **Skills canônicas:** `.agents/skills/`.
- **Claude Code:** mantém adaptadores em `CLAUDE.md`, `.claude/skills/` e `.claude/agents/`.
- **Cursor:** usa `AGENTS.md` e `.agents/skills/` nativamente; atalhos ficam em `.cursor/commands/` e agentes especializados em `.cursor/agents/`.
- **Codex/ChatGPT:** usa `AGENTS.md`, `.agents/skills/` e adaptadores específicos em `.codex/` quando necessários.
- Não crie uma nova cópia de uma skill para um cliente. Atualize primeiro `.agents/skills/` e mantenha qualquer adaptador legado alinhado.
- Se um cliente não disponibilizar subagentes, execute as instruções do agente especializado no contexto principal e gere o mesmo artefato esperado.

## Áreas de alto risco

Solicite confirmação explícita do usuário antes de alterar:

- Schemas de API públicas, payloads de resposta ou comportamento de status codes
- Configuração de deploy, em qualquer das duas pilhas de IaC — domínios, IAM, layers, runtime, packaging (`serverless.ts` ou `bliss-cdk/catalog/`)
- Troca da pilha de IaC de um serviço existente (Serverless Framework ↔ CDK)
- Dependências, lockfiles ou configurações do package manager
- Variáveis de ambiente, secrets ou logging de dados sensíveis
- Migrations de banco de dados em produção
