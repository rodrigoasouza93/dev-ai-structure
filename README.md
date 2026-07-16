# Bliss Task Execution

Projeto agregador de atividades que envolvem múltiplos projetos Bliss
(localizados no diretório de trabalho local, ex.: `~/Documents/bliss/`).

O fluxo de desenvolvimento de features é orquestrado por **skills** em
`.claude/skills/`, com gates de aprovação explícitos entre cada fase.

> As regras completas do projeto estão em [`AGENTS.md`](./AGENTS.md).

## Fluxo de feature em skills

As sete etapas do ciclo de feature são skills. Cada uma só avança quando o
documento da fase anterior contém `Status: APROVADO PELO USUÁRIO`.

```
create-prd  →  create-techspec  →  create-tasks  →  execute-task
                                                          ↓
                            execute-review  ←  execute-bugfix  ←  execute-qa
```

| Fase | Skill | O que faz | Saída em `tasks/prd-[nome]/` |
|------|-------|-----------|------------------------------|
| 1. PRD | `create-prd` | Captura requisitos (faz perguntas antes), foca no O QUÊ/POR QUÊ | `prd.md` |
| 2. TechSpec | `create-techspec` | Traduz o PRD aprovado em arquitetura e decisões técnicas | `techspec.md` |
| 3. Tasks | `create-tasks` | Quebra em tarefas + tarefas de QA, bugfix e review | `tasks.md`, `[num]_task.md` |
| 4. Implementação | `execute-task` | Implementa a próxima tarefa e aciona `@task-reviewer` | código + `tasks.md` atualizado |
| 5. QA | `execute-qa` | Roda testes, a11y e valida contra PRD/TechSpec/Tasks | `qa.md`, `bugs.md` |
| 6. Bugfix | `execute-bugfix` | Corrige os bugs de `bugs.md` com testes de regressão | `bugs.md` atualizado |
| 7. Code Review | `execute-review` | Revisa o `git diff` e a aderência à TechSpec | `codereview.md` |

## Como usar as skills

Há duas formas equivalentes de acionar uma skill no Claude Code:

### 1. Invocação explícita (`/nome`)

Digite o nome da skill como slash command. Ex.:

```
/create-prd  Preciso de um cadastro de responsável legal para dependentes
```

### 2. Acionamento por contexto

Basta descrever a tarefa em linguagem natural — a `description` de cada skill
permite que o modelo acione a skill correta automaticamente:

| Você diz… | Skill acionada |
|-----------|----------------|
| "Vamos começar uma feature de X" / "preciso de um PRD" | `create-prd` |
| "Faça a tech spec dessa feature" (PRD aprovado) | `create-techspec` |
| "Quebre isso em tarefas" (TechSpec aprovada) | `create-tasks` |
| "Implemente a próxima tarefa" (tasks aprovadas) | `execute-task` |
| "Faça o QA dessa feature" | `execute-qa` |
| "Corrija os bugs encontrados" | `execute-bugfix` |
| "Faça o code review" | `execute-review` |

## Gates de aprovação

Regra inegociável do fluxo: **nenhuma fase avança sem aprovação explícita do
usuário na fase anterior.**

- Cada documento nasce com `Status: AGUARDANDO APROVAÇÃO DO USUÁRIO`.
- Ao aprovar, o documento é atualizado para `Status: APROVADO PELO USUÁRIO`.
- `create-techspec` exige `prd.md` aprovado; `create-tasks` exige `prd.md` e
  `techspec.md` aprovados; `execute-task` exige `tasks.md` aprovado.

## Estrutura de uma feature

Todo o trabalho de uma feature fica em `./tasks/prd-[nome-em-kebab-case]/`:

```
tasks/prd-[nome]/
├── prd.md
├── techspec.md
├── tasks.md
├── 1_task.md
├── 2_task.md
├── qa.md
├── bugs.md          (somente se bugs foram encontrados)
├── codereview.md
```

## Skills de apoio

Além do fluxo de feature, o projeto tem skills de padrões e tecnologias
(`code-standards-en`, `nodejs-typescript-conventions`, `fastify-rest-http`,
`serverless-aws-lambda`, `prisma-orm`, `react-frontend-conventions`,
`repo-folder-structure`, `vitest-testing`, `ui-ux-pro-max`, `context7`,
`linear`). A tabela completa, com quando usar cada uma, está em
[`AGENTS.md`](./AGENTS.md).

## Stack dos projetos Bliss

TypeScript • pnpm • Node.js 20+ • AWS Lambda (Serverless Framework) •
Fastify / NestJS • PostgreSQL (Prisma) • Vitest • React + Vite + Ant Design
(intranet). Detalhes e comandos comuns em [`AGENTS.md`](./AGENTS.md).
