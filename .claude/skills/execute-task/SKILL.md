---
name: execute-task
description: >-
  Implementa a próxima tarefa disponível de uma feature, seguindo PRD, TechSpec
  e a lista de tasks aprovados. É a fase de IMPLEMENTAÇÃO do fluxo Bliss
  (PRD → TechSpec → Tasks → Implementação).

  Use quando `tasks.md` contiver `Status: APROVADO PELO USUÁRIO` e o usuário
  pedir para implementar/codar a próxima tarefa. Carrega as skills técnicas
  necessárias, usa Context7, implementa de verdade, aciona @task-reviewer e
  marca a tarefa como concluída em `tasks.md`.

  Não use para QA (execute-qa), correção de bugs (execute-bugfix) ou review
  final (execute-review).
---

# Execução de Tarefa

Você é um assistente IA responsável por implementar as tarefas de forma correta. Você deve identificar a próxima tarefa disponível, realizar a configuração necessária e preparar-se para começar o trabalho **E IMPLEMENTAR**.

<critical>Identifique e carregue as skills necessárias para que a tarefa seja executada com base nas tecnologias utilizadas</critical>
<critical>**VOCÊ DEVE** iniciar a implementação logo após o processo acima.</critical>
<critical>Utilize o Context7 para analisar a documentação da linguagem, frameworks e bibliotecas envolvidas na implementação</critical>
<critical>Após completar a tarefa, marque como completa em tasks.md</critical>

## Informações Fornecidas

## Localização dos Arquivos

- PRD: `./tasks/prd-[nome-funcionalidade]/prd.md`
- Tech Spec: `./tasks/prd-[nome-funcionalidade]/techspec.md`
- Tasks: `./tasks/prd-[nome-funcionalidade]/tasks.md`
- Regras do Projeto: `AGENTS.md`
- Skills do Projeto: `.claude/skills`

## Etapas para Executar

### 1. Configuração Pré-Tarefa

- Ler a definição da tarefa
- Revisar o contexto do PRD
- Verificar requisitos da tech spec
- Entender dependências de tarefas anteriores
- Ler `AGENTS.md`
- Ler os `SKILL.md` aplicáveis em `.claude/skills/<nome>/`
- **Criar uma branch nova no repositório-alvo antes de qualquer alteração** (ver "Convenções de Git" em `AGENTS.md`): sincronizar `main` (`git checkout main && git pull --ff-only`), depois `git checkout -b <tipo>/imp-NNN-slug-curto`. Nunca implementar direto em `main` nem em cima de uma branch de feature antiga não relacionada — confirmar `git merge-base HEAD origin/main` == `git rev-parse origin/main` antes de prosseguir.

<critical>NÃO IMPLEMENTE SEM ANTES CRIAR A BRANCH DA TAREFA A PARTIR DO `main` ATUALIZADO</critical>

### 2. Análise da Tarefa

Analise considerando:

- Objetivos principais da tarefa
- Como a tarefa se encaixa no contexto do projeto
- Alinhamento com regras e padrões do projeto
- Possíveis soluções ou abordagens

### 3. Resumo da Tarefa

```
ID da Tarefa: [ID ou número]
Nome da Tarefa: [Nome ou descrição breve]
Contexto PRD: [Pontos principais do PRD]
Requisitos Tech Spec: [Requisitos técnicos principais]
Dependências: [Lista de dependências]
Objetivos Principais: [Objetivos primários]
Riscos/Desafios: [Riscos ou desafios identificados]
```

### 4. Plano de Abordagem

```
1. [Primeiro passo]
2. [Segundo passo]
3. [Passos adicionais conforme necessário]
```

<critical>NÃO PULE NENHUM PASSO</critical>

### 5. Revisão

1. Execute o agente de review @task-reviewer
2. Ajuste os problemas indicados
3. Não finalize a tarefa até resolver

## Notas Importantes

- Sempre verifique o PRD, tech spec e arquivo de tarefa
- Implemente soluções adequadas **sem usar gambiarras**
- Siga todos os padrões estabelecidos do projeto

<critical>Identifique e carregue as skills necessárias para que a tarefa seja executada com base nas tecnologias utilizadas</critical>
<critical>**VOCÊ DEVE** iniciar a implementação logo após o processo acima.</critical>
<critical>Utilize o Context7 para analisar a documentação da linguagem, frameworks e bibliotecas envolvidas na implementação</critical>
<critical>Após completar a tarefa, marque como completa em tasks.md</critical>
