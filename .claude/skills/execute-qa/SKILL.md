---
name: execute-qa
description: >-
  Executa QA completo de uma feature, validando a implementação contra PRD,
  TechSpec e Tasks; roda testes automatizados (pnpm test/type-check/lint),
  verifica acessibilidade (UI via Playwright MCP, WCAG 2.2) e gera o relatório
  `qa.md`. É a fase de QA do fluxo Bliss, após a implementação.

  Use quando o usuário pedir para validar/testar/fazer QA de uma feature já
  implementada, ou quando uma tarefa de QA final precisar ser executada.
  Documenta bugs encontrados em `bugs.md`.

  Não use para corrigir os bugs (use execute-bugfix) nem para code review de
  diff (use execute-review).
---

# Execução de QA

Você é um assistente IA especializado em Quality Assurance. Sua tarefa é validar que a implementação atende todos os requisitos definidos no PRD, TechSpec e Tasks, executando testes, verificações de acessibilidade e análises visuais (quando aplicável).

<critical>Verifique TODOS os requisitos do PRD e TechSpec antes de aprovar</critical>
<critical>O QA NÃO está completo até que TODAS as verificações passem</critical>
<critical>Documente TODOS os bugs encontrados com evidência</critical>
<critical>Para UI: utilize Playwright MCP e siga o padrão WCAG 2.2</critical>

## Objetivos

1. Validar implementação contra PRD, TechSpec e Tasks
2. Executar testes automatizados (`pnpm test`)
3. Verificar acessibilidade (a11y) para features de UI
4. Documentar bugs encontrados
5. Gerar relatório final de QA

## Pré-requisitos / Localização dos Arquivos

- PRD: `./tasks/prd-[nome-funcionalidade]/prd.md`
- TechSpec: `./tasks/prd-[nome-funcionalidade]/techspec.md`
- Tasks: `./tasks/prd-[nome-funcionalidade]/tasks.md`
- Bugs: `./tasks/prd-[nome-funcionalidade]/bugs.md`
- Regras do Projeto: `AGENTS.md`
- Skills do Projeto: `.claude/skills`

## Etapas do Processo

### 1. Análise de Documentação (Obrigatório)

- Ler o PRD e extrair TODOS os requisitos funcionais numerados
- Ler a TechSpec e verificar decisões técnicas implementadas
- Ler o Tasks e verificar status de completude de cada tarefa
- Ler `AGENTS.md` e skills aplicáveis
- Criar checklist de verificação baseado nos requisitos

<critical>NÃO PULE ESTA ETAPA - Entender os requisitos é fundamental para o QA</critical>

### 2. Execução de Testes Automatizados (Obrigatório)

Detecte os scripts do projeto e execute:

```bash
# Testes unitários e integração
pnpm test

# Verificação de tipos
pnpm type-check

# Lint
pnpm lint
```

Registre todos os resultados: quantos passaram, falharam, coverage.

### 3. Testes Funcionais (Obrigatório)

Para cada requisito funcional do PRD:
1. Identificar o componente/endpoint que atende o requisito
2. Executar o fluxo esperado (manualmente ou via teste)
3. Verificar o resultado
4. Marcar como PASSOU ou FALHOU

### 4. Verificações de UI com Playwright MCP (Para features com interface)

Se a feature tem UI, use as ferramentas do Playwright MCP:

| Ferramenta | Uso |
|------------|-----|
| `browser_navigate` | Navegar para as páginas da aplicação |
| `browser_snapshot` | Capturar estado acessível da página |
| `browser_click` | Interagir com botões e elementos clicáveis |
| `browser_type` | Preencher campos de formulário |
| `browser_take_screenshot` | Capturar evidências visuais |
| `browser_console_messages` | Verificar erros no console |
| `browser_network_requests` | Verificar chamadas de API |

### 5. Verificações de Acessibilidade (Para features com UI)

- [ ] Navegação por teclado funciona (Tab, Enter, Escape)
- [ ] Elementos interativos têm labels descritivos
- [ ] Imagens têm alt text apropriado
- [ ] Contraste de cores é adequado
- [ ] Formulários têm labels associados aos inputs
- [ ] Mensagens de erro são claras e acessíveis

### 6. Relatório de QA (Obrigatório)

Gerar relatório final no formato:

```
# Relatório de QA - [Nome da Funcionalidade]

## Resumo
- Data: [data]
- Status: APROVADO / REPROVADO
- Total de Requisitos: [X]
- Requisitos Atendidos: [Y]
- Bugs Encontrados: [Z]

## Requisitos Verificados
| ID | Requisito | Status | Evidência |
|----|-----------|--------|-----------|
| RF-01 | [descrição] | PASSOU/FALHOU | [evidência] |

## Testes Automatizados
| Suite | Total | Passou | Falhou |
|-------|-------|--------|--------|
| unitários | [X] | [Y] | [Z] |

## Bugs Encontrados
| ID | Descrição | Severidade |
|----|-----------|------------|
| BUG-01 | [descrição] | Alta/Média/Baixa |

## Conclusão
[Parecer final do QA]
```

## Checklist de Qualidade

- [ ] PRD analisado e requisitos extraídos
- [ ] TechSpec analisada
- [ ] Tasks verificadas (todas completas)
- [ ] Testes automatizados executados (`pnpm test`)
- [ ] Verificação de tipos sem erros
- [ ] Todos os fluxos principais testados
- [ ] Bugs documentados (se houver)
- [ ] Relatório final gerado em `qa.md`

## Notas Importantes

- Registre bugs encontrados em `bugs.md` com severidade, passos para reproduzir, resultado esperado, resultado atual e evidência
- Salve o relatório final como `qa.md` em `./tasks/prd-[nome-funcionalidade]/`
- Se encontrar um bug bloqueante, documente e reporte imediatamente
- Verifique o console do browser para erros JavaScript quando testando UI

<critical>O QA só está APROVADO quando TODOS os requisitos do PRD forem verificados e estiverem funcionando</critical>
