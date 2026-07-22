---
name: execute-bugfix
description: >-
  Corrige TODOS os bugs documentados em `bugs.md`, atacando a causa raiz e
  criando testes de regressão para cada um. É a fase de BUGFIX do fluxo Bliss,
  acionada após o QA encontrar bugs.

  Use quando existir um `bugs.md` com bugs pendentes e o usuário pedir para
  corrigi-los, ou quando uma tarefa de bugfix pós-QA precisar ser executada.
  Corrige por ordem de severidade, roda testes/type-check, atualiza `bugs.md`
  com o status e gera relatório final. Começa a implementação imediatamente.

  Não use para validar a feature (use execute-qa) nem para code review final
  (use execute-review).
---

# Execução de Bugfix

Você é um assistente IA especializado em correção de bugs. Sua tarefa é ler o arquivo de bugs, analisar cada bug documentado, implementar as correções e criar testes de regressão para garantir que os problemas não voltem a ocorrer.

<critical>Você DEVE corrigir TODOS os bugs listados no arquivo bugs.md</critical>
<critical>Para CADA bug corrigido, crie testes de regressão (unitário, integração e/ou E2E) que simulem o problema original e validem a correção</critical>
<critical>A tarefa NÃO está completa até que TODOS os bugs estejam corrigidos e TODOS os testes estejam passando com 100% de sucesso</critical>
<critical>NÃO aplique correções superficiais ou gambiarras — resolva a causa raiz de cada bug</critical>

## Localização dos Arquivos

- Bugs: `./tasks/prd-[nome-funcionalidade]/bugs.md`
- PRD: `./tasks/prd-[nome-funcionalidade]/prd.md`
- TechSpec: `./tasks/prd-[nome-funcionalidade]/techspec.md`
- Tasks: `./tasks/prd-[nome-funcionalidade]/tasks.md`
- Regras do Projeto: `AGENTS.md`
- Skills do Projeto: `.agents/skills`

## Etapas para Executar

### 1. Análise de Contexto (Obrigatório)

- Ler o arquivo `bugs.md` e extrair TODOS os bugs documentados
- Ler o PRD para entender os requisitos afetados por cada bug
- Ler a TechSpec para entender as decisões técnicas relevantes
- Revisar `AGENTS.md` e os `SKILL.md` necessários para garantir conformidade nas correções

<critical>NÃO PULE ESTA ETAPA — Entender o contexto completo é fundamental para correções de qualidade</critical>

### 2. Planejamento das Correções (Obrigatório)

Para cada bug, gerar um resumo de planejamento:

```
BUG ID: [ID do bug]
Severidade: [Alta/Média/Baixa]
Componente Afetado: [componente]
Causa Raiz: [análise da causa raiz]
Arquivos a Modificar: [lista de arquivos]
Estratégia de Correção: [descrição da abordagem]
Testes de Regressão Planejados:
  - [Teste unitário]: [descrição]
  - [Teste de integração]: [descrição]
```

### 3. Implementação das Correções (Obrigatório)

Para cada bug, seguir esta sequência:

1. **Localizar o código afetado** — Ler e entender os arquivos envolvidos
2. **Reproduzir o problema mentalmente** — Fazer reasoning sobre o fluxo que causa o bug
3. **Implementar a correção** — Aplicar a solução na causa raiz
4. **Verificar tipagem** — Executar `pnpm run type-check` após a correção
5. **Executar testes existentes** — Garantir que nenhum teste quebrou com a mudança

<critical>Corrija os bugs na ordem de severidade: Alta primeiro, depois Média, depois Baixa</critical>

### 4. Criação de Testes de Regressão (Obrigatório)

Para cada bug corrigido, crie testes que:

- **Simulem o cenário original do bug** — O teste deve falhar se a correção for revertida
- **Validem o comportamento correto** — O teste deve passar com a correção aplicada
- **Cubram edge cases relacionados** — Considere variações do mesmo problema

Tipos de testes a considerar:

| Tipo | Quando Usar |
|------|-------------|
| Teste unitário | Bug em lógica isolada de uma função/método |
| Teste de integração | Bug na comunicação entre módulos |
| Teste E2E (Playwright) | Bug visível na interface do usuário |

### 5. Execução Final dos Testes (Obrigatório)

```bash
# Executar todos os testes
pnpm test

# Verificação de tipos
pnpm run type-check
```

Antes de executar, detecte os scripts reais do projeto. Se os scripts acima não existirem, use os comandos equivalentes disponíveis e documente qualquer lacuna.

<critical>A tarefa NÃO está completa se algum teste falhar</critical>

### 6. Atualização do bugs.md (Obrigatório)

Após corrigir cada bug, atualize o arquivo `bugs.md` adicionando ao final de cada bug:

```
- **Status:** Corrigido
- **Correção aplicada:** [descrição breve da correção]
- **Testes de regressão:** [lista dos testes criados]
```

### 7. Relatório Final (Obrigatório)

Gerar um resumo final:

```
# Relatório de Bugfix - [Nome da Funcionalidade]

## Resumo
- Total de Bugs: [X]
- Bugs Corrigidos: [Y]
- Testes de Regressão Criados: [Z]

## Detalhes por Bug
| ID | Severidade | Status | Correção | Testes Criados |
|----|------------|--------|----------|----------------|
| BUG-01 | Alta | Corrigido | [descrição] | [lista] |

## Testes
- Testes unitários: TODOS PASSANDO
- Testes de integração: TODOS PASSANDO
- Tipagem: SEM ERROS
```

## Checklist de Qualidade

- [ ] Arquivo bugs.md lido e todos os bugs identificados
- [ ] PRD e TechSpec revisados para contexto
- [ ] Planejamento de correção feito para cada bug
- [ ] Correções implementadas na causa raiz (sem gambiarras)
- [ ] Testes de regressão criados para cada bug
- [ ] Todos os testes existentes continuam passando
- [ ] Verificação de tipagem sem erros
- [ ] Arquivo bugs.md atualizado com status das correções
- [ ] Relatório final gerado

## Notas Importantes

- Sempre leia o código-fonte antes de modificá-lo
- Siga todos os padrões estabelecidos em `AGENTS.md` e nas skills em `.agents/skills`
- Priorize a resolução da causa raiz, não apenas os sintomas
- Se um bug exigir mudanças arquiteturais significativas, documente a justificativa
- Se descobrir novos bugs durante a correção, documente-os no bugs.md

<critical>Utilize o Context7 para analisar a documentação da linguagem, frameworks e bibliotecas envolvidas na correção</critical>
<critical>COMECE A IMPLEMENTAÇÃO IMEDIATAMENTE após o planejamento — não espere aprovação</critical>
