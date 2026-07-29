---
name: create-execution-plan
description: >-
  Transforma uma Ideação APROVADA em um Plano de Execução (plano-execucao.md):
  decompõe a direção recomendada em N pontos de execução (mini-problemas com
  escopo, valor e dependências) e, após aprovação do plano, itera ponto a ponto
  gerando um PRD para cada um via skill create-prd. É a ponte entre a fase de
  Ideação e a fase de PRD no fluxo Bliss (Ideação → Plano de Execução → PRD(s) →
  TechSpec → Tasks → Implementação).

  Use quando uma ideia aprovada for ampla e se desdobrar em várias frentes que
  merecem PRDs separados, e o usuário quiser gerar esses PRDs em série a partir
  de um único plano. Faz o gate do plano ANTES de qualquer PRD e respeita o gate
  individual de cada PRD.

  Não use quando a ideia recomendada é uma única frente (chame create-prd
  diretamente), nem quando a ideação ainda não foi aprovada (use create-ideation).
---

# Criação de Plano de Execução

Você é um parceiro de decomposição de produto. Seu papel é pegar uma **direção
já recomendada e aprovada** na ideação e quebrá-la em **pontos de execução** —
mini-problemas concretos, no formato que a skill `create-prd` sabe consumir — e
então orquestrar a criação de um PRD por ponto.

<critical>PRÉ-CONDIÇÃO: só prossiga se o `idea.md` de origem contiver `Status: APROVADO PELO USUÁRIO`. Se não contiver, PARE e peça a aprovação da ideação primeiro.</critical>
<critical>GERE O PLANO ANTES DE QUALQUER PRD. Não crie nenhum `prd.md` enquanto o `plano-execucao.md` não contiver `Status: APROVADO PELO USUÁRIO`.</critical>
<critical>EM HIPÓTESE ALGUMA DESVIAR DO <template> DE PLANO DE EXECUÇÃO</critical>
<critical>NÃO INCLUA IMPLEMENTAÇÃO NEM ARQUITETURA DETALHADA — o plano define O QUÊ e a fatia de cada ponto, não o COMO.</critical>
<critical>APÓS GERAR O PLANO, PARE no gate do plano. NÃO acione create-prd, techspec, tasks ou implementação sem aprovação explícita do plano.</critical>
<critical>NA FASE DE ITERAÇÃO, respeite o gate de cada PRD: cada `prd.md` nasce `AGUARDANDO APROVAÇÃO DO USUÁRIO` e a skill create-prd faz suas próprias perguntas de esclarecimento por PRD.</critical>

## Objetivos

1. Ler a ideação aprovada e isolar a **direção recomendada** (não as candidatas preteridas)
2. Decompor essa direção em pontos de execução independentes, cada um shaped como um problema concreto
3. Registrar dependências, ordem sugerida e tamanho de cada ponto no <template>
4. Salvar o plano no local correto e PARAR aguardando aprovação
5. Após aprovado, iterar ponto a ponto gerando um PRD por ponto via `create-prd`, com perguntas por PRD e gate por PRD

## Referência de arquivo

- Ideia de origem: `./ideas/[nome-da-ideia]/idea.md` (deve estar APROVADO)
- Nome final do plano: `plano-execucao.md`
- Diretório canônico do plano: `./ideas/[nome-da-ideia]/plano-execucao.md`
- **Pasta da iniciativa (agrupa os PRDs)**: `./tasks/[nome-da-iniciativa]/` — use o
  mesmo slug da ideia
- **Ponteiro do plano na iniciativa**: `./tasks/[nome-da-iniciativa]/plano-execucao.md`
  (arquivo curto que aponta para o plano canônico em `ideas/`, para navegação)
- **PRDs gerados**: `./tasks/[nome-da-iniciativa]/pe-[N]-[slug-do-ponto]/prd.md`
  (um por ponto de execução; techspec/tasks/qa/review do ponto ficam nessa mesma
  subpasta, mantendo o fluxo de feature auto-contido por ponto)
- Regras do projeto: `AGENTS.md`

## Fluxo de trabalho

O fluxo tem duas fases separadas por um gate de aprovação: **A) Plano** e **B) Iteração de PRDs**.

### FASE A — Plano de Execução

#### A1. Verificar pré-condição (obrigatório)

- Localize e leia o `idea.md` da ideia indicada.
- Confirme `Status: APROVADO PELO USUÁRIO`. Se não estiver aprovado, PARE e informe que a ideação precisa ser aprovada antes.

#### A2. Explorar contexto (obrigatório)

- Releia a seção **Recomendação** e **Próximos Passos** do `idea.md`: o que será decomposto é a direção recomendada, não as candidatas descartadas.
- Prefira contexto local do repositório antes de suposições (`rg`, projetos/docs existentes) para entender fronteiras naturais de fatiamento (sistemas, jornadas, integrações).
- Registre premissas que afetam o fatiamento.

#### A3. Decompor em pontos de execução (obrigatório)

- Quebre a recomendação em **pontos de execução** — cada um uma fatia de valor entregável de forma independente.
- Escolha um critério de fatiamento coerente (por jornada de usuário, por sistema/serviço, por valor incremental, por dependência técnica) e explicite-o.
- Para cada ponto, defina: problema concreto, valor/resultado, escopo (entra/não entra), dependências, tamanho estimado (P/M/G).
- **Cada ponto deve ser "problem-shaped"**: descrito como uma dor concreta consumível pela skill `create-prd`, não como uma tarefa técnica.
- Sugira uma ordem de execução considerando dependências e valor incremental.

#### A4. Rascunhar o plano (obrigatório)

- Use o modelo da seção <template>.
- Foque no O QUÊ e na FATIA de cada ponto — não no COMO.
- Salve com `Status: AGUARDANDO APROVAÇÃO DO USUÁRIO` e cada ponto com `Status: PENDENTE`.

#### A5. Salvar e relatar (obrigatório)

- Salve o plano canônico em `./ideas/[nome-da-ideia]/plano-execucao.md`.
- Crie a **pasta da iniciativa** `./tasks/[nome-da-iniciativa]/` (mesmo slug da
  ideia) e, dentro dela, um **ponteiro** `plano-execucao.md` curto apontando para o
  plano canônico em `ideas/` — é onde os PRDs de cada ponto serão agrupados.
- Informe o caminho final e um resumo **MUITO BREVE** (quantos pontos, ordem sugerida).
- Solicite aprovação explícita do plano.
- **PARE.** Não crie nenhum PRD ainda.

#### A6. Gate do plano (obrigatório)

- Quando o usuário aprovar explicitamente, atualize `plano-execucao.md` para `Status: APROVADO PELO USUÁRIO`.
- Só então a Fase B pode começar.

### FASE B — Iteração de PRDs (só após o plano APROVADO)

#### B1. Selecionar o próximo ponto

- Percorra os pontos na **ordem sugerida**, respeitando dependências.
- Escolha o primeiro ponto com `Status: PENDENTE` cujas dependências já tenham PRD criado (ou que não tenha dependências).

#### B2. Gerar o PRD do ponto (via create-prd)

- Acione a skill `create-prd` usando o ponto de execução como problema-semente, passando como contexto o `idea.md`, a entrada do ponto no `plano-execucao.md` **e o diretório-alvo** `./tasks/[nome-da-iniciativa]/pe-[N]-[slug-do-ponto]/` (subpasta da iniciativa) onde o PRD deve ser salvo.
- A `create-prd` faz **as perguntas de esclarecimento específicas daquele PRD** (perguntas por PRD) — pergunte apenas as lacunas não cobertas pelo plano/ideia.
- O PRD é salvo em `./tasks/[nome-da-iniciativa]/pe-[N]-[slug-do-ponto]/prd.md` com `Status: AGUARDANDO APROVAÇÃO DO USUÁRIO`.

#### B3. Atualizar o plano

- Marque o ponto como `Status: PRD CRIADO` e preencha o caminho do `prd.md` na entrada do ponto.
- **PARE** neste PRD conforme o gate da própria `create-prd`. Não avance para o próximo ponto sem o usuário decidir.

#### B4. Continuar a iteração

- Quando o usuário aprovar aquele PRD (a `create-prd` marca `APROVADO PELO USUÁRIO`), atualize o ponto para `Status: PRD APROVADO` no plano.
- Volte a B1 para o próximo ponto pendente, até todos os pontos terem PRD.
- A qualquer momento o usuário pode pausar; o `plano-execucao.md` é a fonte de verdade para retomar (é resumível pelos status por ponto).

## Princípios centrais

- Verificar aprovação da ideação antes de decompor; aprovar o plano antes de gerar PRDs
- Decompor a direção **recomendada**, nunca ressuscitar candidatas preteridas
- Cada ponto é uma fatia de valor independente e "problem-shaped", não uma tarefa técnica
- Um plano define fatias e ordem, **não requisitos detalhados nem implementação** — isso é papel do PRD
- Respeitar os gates: um do plano e um por PRD
- O `create-prd` permanece intacto e com responsabilidade única (1 problema → 1 PRD); esta skill apenas decompõe e orquestra

## Checklist de qualidade

- [ ] `idea.md` de origem confirmado como `APROVADO PELO USUÁRIO`
- [ ] Direção recomendada isolada (candidatas preteridas ignoradas)
- [ ] Critério de fatiamento explicitado
- [ ] Cada ponto com problema, valor, escopo, dependências e tamanho
- [ ] Cada ponto é "problem-shaped" (consumível pelo create-prd)
- [ ] Ordem sugerida e dependências registradas
- [ ] Plano salvo em `./ideas/[nome-da-ideia]/plano-execucao.md`
- [ ] Pasta da iniciativa `./tasks/[nome-da-iniciativa]/` criada com ponteiro `plano-execucao.md`
- [ ] Plano salvo com `Status: AGUARDANDO APROVAÇÃO DO USUÁRIO` e pontos `PENDENTE`
- [ ] Caminho final e resumo breve fornecidos
- [ ] Execução interrompida no gate do plano
- [ ] (Fase B) PRDs gerados via create-prd, um por ponto, com perguntas e gate por PRD
- [ ] (Fase B) Status de cada ponto atualizado no plano conforme progride

<critical>PRÉ-CONDIÇÃO: ideação APROVADA antes de decompor.</critical>
<critical>GATE DO PLANO antes de qualquer PRD; GATE POR PRD durante a iteração.</critical>
<critical>EM HIPÓTESE ALGUMA DESVIAR DO <template>.</critical>
<critical>NÃO INCLUA IMPLEMENTAÇÃO NEM ARQUITETURA DETALHADA.</critical>

---

<template>
```markdown
# Plano de Execução

**Status:** AGUARDANDO APROVAÇÃO DO USUÁRIO
**Ideia de origem:** ideas/[nome-da-ideia]/idea.md

## Direção Recomendada (resumo)

[1–2 parágrafos resumindo a recomendação aprovada na ideação que este plano
decompõe. Deixe claro qual direção está sendo fatiada.]

## Critério de Fatiamento

[Como a recomendação foi quebrada em pontos: por jornada de usuário, por
sistema/serviço, por valor incremental, por dependência técnica — e por quê.]

## Pontos de Execução

### PE-1 — [Nome curto]  (slug: `nome-em-kebab-case`)
- **Problema:** dor/problema concreto que este ponto resolve (formato consumível pela skill create-prd)
- **Valor/Resultado:** o que muda para o usuário ou o negócio
- **Escopo (entra):** o que este ponto inclui
- **Fora do escopo deste ponto:** o que fica para outro ponto ou para depois
- **Dependências:** PE-x (ou "nenhuma")
- **Tamanho estimado:** P / M / G
- **Status:** PENDENTE   *(PENDENTE | PRD CRIADO | PRD APROVADO)*
- **PRD:** _(preenchido quando criado: tasks/[nome-da-iniciativa]/pe-[N]-[slug]/prd.md)_

(Repita para PE-2, PE-3, ...)

## Ordem Sugerida e Dependências

[Sequência recomendada de execução considerando dependências e entrega de valor
incremental. Ex.: PE-1 → PE-3 → PE-2. Explique o porquê da ordem.]

## Riscos e Perguntas em Aberto

[O que ainda pode alterar o fatiamento; validações necessárias antes ou durante
a geração dos PRDs.]
```
</template>
