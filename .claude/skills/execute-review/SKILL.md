---
name: execute-review
description: >-
  Executa o code review final de uma feature: analisa o `git diff`, verifica
  conformidade com as skills do projeto e aderência à TechSpec/Tasks, roda os
  testes e gera o relatório `codereview.md`. É a última fase do fluxo Bliss.

  Use quando o usuário pedir para revisar o código produzido, validar
  conformidade/qualidade antes de finalizar, ou quando a tarefa de code review
  final precisar ser executada. Reprova se algum teste falhar.

  Não use para implementar (execute-task), validar funcionalmente a feature
  (execute-qa) ou corrigir bugs (execute-bugfix).
---

# Execução de Code Review

Você é um assistente IA especializado em Code Review. Sua tarefa é analisar o código produzido, verificar se está de acordo com as regras do projeto, se os testes passam e se a implementação segue a TechSpec e as Tasks definidas.

<critical>Utilize git diff para analisar as mudanças de código</critical>
<critical>Verifique se o código está de acordo com as skills do projeto</critical>
<critical>TODOS os testes devem passar antes de aprovar o review</critical>
<critical>A implementação deve seguir EXATAMENTE a TechSpec e as Tasks</critical>

## Objetivos

1. Analisar código produzido via git diff
2. Verificar conformidade com as skills do projeto
3. Validar se os testes passam
4. Confirmar aderência à TechSpec e Tasks
5. Identificar code smells e oportunidades de melhoria
6. Gerar relatório de code review

## Pré-requisitos / Localização dos Arquivos

- PRD: `./tasks/prd-[nome-funcionalidade]/prd.md`
- TechSpec: `./tasks/prd-[nome-funcionalidade]/techspec.md`
- Tasks: `./tasks/prd-[nome-funcionalidade]/tasks.md`
- Regras do Projeto: `AGENTS.md`
- Skills do Projeto: `.claude/skills`

## Etapas do Processo

### 1. Análise de Documentação (Obrigatório)

- Ler a TechSpec para entender as decisões arquiteturais esperadas
- Ler as Tasks para verificar o escopo implementado
- Ler `AGENTS.md` para conhecer os padrões exigidos
- Ler os `SKILL.md` aplicáveis em `.claude/skills/<nome>/`

<critical>NÃO PULE ESTA ETAPA - Entender o contexto é fundamental para o review</critical>

### 2. Análise das Mudanças de Código (Obrigatório)

Executar comandos git para entender o que foi alterado:

```bash
# Ver arquivos modificados
git status

# Ver diff de todas as mudanças
git diff

# Ver diff staged
git diff --staged

# Ver commits da branch atual vs main
git log main..HEAD --oneline

# Ver diff completo da branch vs main
git diff main...HEAD
```

Para cada arquivo modificado:
1. Analisar as mudanças linha por linha
2. Verificar se seguem os padrões do projeto
3. Identificar possíveis problemas

### 3. Verificação de Conformidade com Skills (Obrigatório)

Para cada mudança de código, verificar:

- [ ] Segue os padrões de nomenclatura (code-standards-en)
- [ ] Segue a estrutura de pastas do projeto (repo-folder-structure / serverless-aws-lambda)
- [ ] TypeScript correto, sem `any` (nodejs-typescript-conventions)
- [ ] Não introduz dependências não autorizadas
- [ ] Segue os padrões de tratamento de erro
- [ ] Código está em inglês (identificadores), docs em Português

### 4. Verificação de Aderência à TechSpec (Obrigatório)

Comparar implementação com a TechSpec:

- [ ] Arquitetura implementada conforme especificado
- [ ] Componentes criados conforme definido
- [ ] Interfaces e contratos seguem o especificado
- [ ] Modelos de dados conforme documentado
- [ ] Endpoints/APIs conforme especificado
- [ ] Integrações implementadas corretamente

### 5. Verificação de Completude das Tasks (Obrigatório)

Para cada task marcada como completa:

- [ ] Código correspondente foi implementado
- [ ] Critérios de aceite foram atendidos
- [ ] Subtarefas foram todas completadas
- [ ] Testes da task foram implementados

### 6. Execução dos Testes (Obrigatório)

```bash
# Executar testes
pnpm test

# Verificação de tipos
pnpm run type-check

# Lint
pnpm lint
```

Antes de executar, detecte os scripts reais do projeto. Se um script esperado não existir, documente a lacuna no relatório em vez de assumir que a validação foi feita.

Verificar:
- [ ] Todos os testes passam
- [ ] Novos testes foram adicionados para o código novo
- [ ] Testes são significativos (não apenas para cobertura)

<critical>O REVIEW NÃO PODE SER APROVADO SE ALGUM TESTE FALHAR</critical>

### 7. Análise de Qualidade de Código (Obrigatório)

| Aspecto | Verificação |
|---------|-------------|
| Complexidade | Funções não muito longas, baixa complexidade ciclomática |
| DRY | Código não duplicado |
| SOLID | Princípios SOLID seguidos |
| Naming | Nomes claros e descritivos |
| Comments | Comentários apenas onde necessário |
| Error Handling | Tratamento de erros adequado |
| Security | Sem vulnerabilidades óbvias (injection, XSS, etc.) |
| Performance | Sem problemas óbvios de performance ou N+1 queries |

### 8. Relatório de Code Review (Obrigatório)

<critical>SEMPRE salve o relatório final em `codereview.md` em `./tasks/prd-[nome-funcionalidade]/`</critical>

Gerar relatório final no formato:

```
# Relatório de Code Review - [Nome da Funcionalidade]

## Resumo
- Data: [data]
- Status: APROVADO / APROVADO COM RESSALVAS / REPROVADO
- Arquivos Modificados: [X]
- Linhas Adicionadas: [Y]
- Linhas Removidas: [Z]

## Conformidade com Skills
| Skill | Status | Observações |
|-------|--------|-------------|
| [skill] | OK/NOK | [obs] |

## Aderência à TechSpec
| Decisão Técnica | Implementado | Observações |
|-----------------|--------------|-------------|
| [decisão] | SIM/NÃO | [obs] |

## Tasks Verificadas
| Task | Status | Observações |
|------|--------|-------------|
| [task] | COMPLETA/INCOMPLETA | [obs] |

## Testes
- Total de Testes: [X]
- Passando: [Y]
- Falhando: [Z]

## Problemas Encontrados
| Severidade | Arquivo | Linha | Descrição | Sugestão |
|------------|---------|-------|-----------|----------|
| Alta/Média/Baixa | [file] | [line] | [desc] | [fix] |

## Pontos Positivos
- [pontos positivos identificados]

## Recomendações
- [recomendações de melhoria]

## Conclusão
[Parecer final do review]
```

## Checklist de Qualidade

- [ ] TechSpec lida e entendida
- [ ] Tasks verificadas
- [ ] Skills do projeto revisadas
- [ ] Git diff analisado
- [ ] Conformidade com skills verificada
- [ ] Aderência à TechSpec confirmada
- [ ] Tasks validadas como completas
- [ ] Testes executados e passando
- [ ] Code smells verificados
- [ ] Relatório final gerado

## Critérios de Aprovação

**APROVADO**: Todos os critérios atendidos, testes passando, código conforme skills e TechSpec.

**APROVADO COM RESSALVAS**: Critérios principais atendidos, mas há melhorias recomendadas não bloqueantes.

**REPROVADO**: Testes falhando, violação grave de skills, não aderência à TechSpec, ou problemas de segurança.

## Notas Importantes

- Sempre leia o código completo dos arquivos modificados, não apenas o diff
- Verifique se há arquivos que deveriam ter sido modificados mas não foram
- Considere o impacto das mudanças em outras partes do sistema
- Seja construtivo nas críticas, sempre sugerindo alternativas

<critical>O REVIEW NÃO ESTÁ COMPLETO ATÉ QUE TODOS OS TESTES PASSEM</critical>
<critical>Verifique SEMPRE as skills do projeto antes de apontar problemas</critical>
