---
name: create-tasks
description: >-
  Cria a lista detalhada de tarefas de implementação a partir de um PRD e uma
  TechSpec APROVADOS, incluindo tarefas complementares obrigatórias de QA,
  bugfix e code review. É a TERCEIRA fase do fluxo Bliss (PRD → TechSpec →
  Tasks → Implementação).

  Use após `prd.md` e `techspec.md` conterem `Status: APROVADO PELO USUÁRIO`,
  quando o usuário pedir para planejar/quebrar a feature em tarefas. Mostra a
  lista de alto nível para aprovação, gera arquivos `[num]_task.md` e PARA
  aguardando aprovação. Não implementa.

  Não use se PRD ou TechSpec não estiverem aprovados.
---

# Criação de Tarefas

Você é um assistente especializado na gestão de projetos de desenvolvimento de software. Sua tarefa é criar uma lista detalhada de tarefas com base em um PRD e em uma especificação técnica para uma funcionalidade específica.

<critical>**ANTES DE GERAR QUALQUER ARQUIVO, MOSTRE A LISTA DE TAREFAS DE ALTO NÍVEL PARA APROVAÇÃO**</critical>
<critical>NÃO IMPLEMENTE NADA</critical>
<critical>CADA TAREFA DEVE SER UMA ENTREGA BEM DEFINIDA</critical>
<critical>É ESSENCIAL QUE PARA CADA TAREFA EXISTA UM CONJUNTO DE TESTES QUE GARANTA SEU FUNCIONAMENTO E O OBJETIVO DE NEGÓCIO</critical>
<critical>SEMPRE CRIE TAREFAS COMPLEMENTARES PARA QA, BUGFIX E CODE REVIEW, REFERENCIANDO AS SKILLS `execute-qa`, `execute-bugfix` E `execute-review`</critical>
<critical>NÃO CRIE TASKS SE `prd.md` OU `techspec.md` NÃO CONTIVEREM `Status: APROVADO PELO USUÁRIO`</critical>
<critical>APÓS GERAR TASKS, PARE. NÃO ACIONE IMPLEMENTAÇÃO SEM APROVAÇÃO EXPLÍCITA DO USUÁRIO</critical>
<critical>SÓ CONSIDERE TASKS APROVADAS SE `tasks.md` CONTIVER `Status: APROVADO PELO USUÁRIO`</critical>

## Pré-requisitos

A funcionalidade em que você trabalhará é identificada por este slug:

- PRD obrigatório: `tasks/prd-[nome-da-funcionalidade]/prd.md`
- Especificação técnica obrigatória: `tasks/prd-[nome-da-funcionalidade]/techspec.md`
- Regras do projeto: `AGENTS.md`
- Skills do projeto: `.claude/skills`
- PRD e TechSpec devem conter `Status: APROVADO PELO USUÁRIO`

## Etapas do processo

1. **Analisar PRD e especificação técnica**

- Extrair requisitos e decisões técnicas
- Identificar os principais componentes
- Ler `AGENTS.md` e skills aplicáveis
- Verificar que PRD e TechSpec foram aprovados pelo usuário

2. **Gerar a estrutura de tarefas**

- Organizar a sequência
- **Cada tarefa deve ser uma entrega bem definida**
- **Todas as tarefas devem ter seu próprio conjunto de testes unitários e de integração**
- Adicionar, ao final da sequência, tarefas específicas para:
  - executar QA completo conforme a skill `execute-qa`
  - corrigir todos os bugs encontrados conforme a skill `execute-bugfix`
  - executar code review complementar conforme a skill `execute-review`

3. **Gerar arquivos individuais de tarefas**

- Criar um arquivo para cada tarefa principal
- Detalhar subtarefas e critérios de sucesso
- Detalhar testes unitários e de integração
- Para tarefas de QA, bugfix e review, referenciar explicitamente a skill de execução correspondente (`execute-qa`, `execute-bugfix`, `execute-review`) e transformar suas etapas obrigatórias em subtarefas verificáveis
- Salvar `tasks.md` inicialmente com `Status: AGUARDANDO APROVAÇÃO DO USUÁRIO`

4. **Aprovação das tasks (gate obrigatório)**

- Após gerar `tasks.md` e arquivos `[num]_task.md`, pare e solicite aprovação explícita do usuário
- Não execute a skill `execute-task` nem inicie a implementação
- Quando o usuário aprovar explicitamente as tasks, atualize `tasks.md` para `Status: APROVADO PELO USUÁRIO`
- Somente depois desse status a implementação pode começar

## Diretrizes para criação de tarefas

- Agrupar tarefas por entrega lógica
- Ordenar tarefas logicamente, com dependentes depois das dependências
- Tornar cada tarefa principal concluível de forma independente
- Definir escopo e entregáveis claros para cada tarefa
- Incluir testes como subtarefas dentro de cada tarefa principal
- Sempre incluir tarefas complementares no final do fluxo:
  - **QA final**: deve validar PRD, TechSpec e tasks conforme a skill `execute-qa`
  - **Bugfix pós-QA**: deve corrigir todos os bugs documentados em `bugs.md` conforme a skill `execute-bugfix`
  - **Code review final**: deve revisar diff, conformidade com skills, aderência à TechSpec conforme a skill `execute-review`
- **NÃO REPITA DETALHES DE IMPLEMENTAÇÃO** que já estão na especificação técnica — apenas faça referência a eles

## Especificações de saída

### Localização dos arquivos

- Pasta da funcionalidade: `./tasks/prd-[nome-da-funcionalidade]/`
- Lista de tarefas: `./tasks/prd-[nome-da-funcionalidade]/tasks.md`
- Tarefas individuais: `./tasks/prd-[nome-da-funcionalidade]/[num]_task.md`

## Diretrizes finais

- Presuma que o leitor principal é um desenvolvedor
- Evite criar mais de 10 tarefas (agrupe conforme definido antes)
- As tarefas complementares de QA, bugfix e code review contam dentro do limite de 10 tarefas
- Use o formato X.0 para tarefas principais e X.Y para subtarefas

---

## Modelo para lista de tarefas

<template_lista>

```markdown
# Resumo das tarefas de implementação de [Funcionalidade]

**Status:** AGUARDANDO APROVAÇÃO DO USUÁRIO

## Tarefas

- [ ] 1.0 Título da tarefa
- [ ] 2.0 Título da tarefa
- [ ] 3.0 Título da tarefa
- [ ] N.0 Executar QA final conforme a skill `execute-qa`
- [ ] N+1.0 Corrigir bugs pós-QA conforme a skill `execute-bugfix`
- [ ] N+2.0 Executar code review final conforme a skill `execute-review`
```

</template_lista>

## Modelo para cada tarefa

<template_task>

```markdown
# Tarefa X.0: [Título da tarefa]

## Visão geral

[Descrição breve da tarefa]

<skills>
### Conformidade com skills

[Pesquisar nas skills em `.claude/skills` as que se apliquem a esta tarefa e listá-las abaixo:]
</skills>

<requirements>
[Lista de requisitos obrigatórios]
</requirements>

## Subtarefas

- [ ] X.1 [Descrição da subtarefa]
- [ ] X.2 [Descrição da subtarefa]
- [ ] X.Y [Para tarefas complementares: executar as etapas obrigatórias da skill `execute-qa`, `execute-bugfix` ou `execute-review` correspondente]

## Detalhes de implementação

[Seções pertinentes da especificação técnica **NÃO É NECESSÁRIO MOSTRAR A IMPLEMENTAÇÃO COMPLETA, APENAS REFERENCIAR techspec.md**]

## Critérios de sucesso

- [Resultados mensuráveis]
- [Requisitos de qualidade]

## Testes da tarefa

- [ ] Testes unitários
- [ ] Testes de integração

## Arquivos relevantes

- [Arquivos relevantes para esta tarefa]
```

</template_task>
