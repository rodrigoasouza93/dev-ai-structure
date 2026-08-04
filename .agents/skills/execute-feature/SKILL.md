---
name: execute-feature
description: >-
  Executa continuamente todas as tarefas aprovadas de uma feature Bliss dentro
  da sessão atual, reutilizando as skills de implementação, QA, bugfix e review.
  Cria ou reutiliza uma única branch da feature, revisa e autocorrige cada task,
  roda as verificações aplicáveis, marca a task como concluída e cria um commit
  local antes de seguir. Use quando `prd.md`, `techspec.md` e `tasks.md` estiverem
  APROVADOS PELO USUÁRIO e o usuário pedir para executar todas as tasks, continuar
  até finalizar a feature ou trabalhar em loop. Não faz push nem abre PR.
---

# Executar feature completa

Executar todas as tasks aprovadas sem parar entre tarefas bem-sucedidas. Tratar
`tasks.md` e o Git como estado de retomada; não criar manifesto ou cópia paralela.

## Regras essenciais

- Ler `AGENTS.md`, `prd.md`, `techspec.md`, `tasks.md` e os arquivos
  `[num]_task.md` antes de começar.
- Exigir `Status: APROVADO PELO USUÁRIO` em PRD, TechSpec e Tasks.
- Usar uma única branch para toda a feature. Não criar uma branch por task.
- Preservar mudanças preexistentes e nunca incluí-las acidentalmente em commits.
- Executar review e verificações antes de concluir e commitar cada task.
- Corrigir falhas automaticamente e verificar novamente.
- Não fazer push, não abrir PR e não executar merge.
- Não encerrar a resposta após uma task: seguir até todas estarem concluídas ou
  existir um bloqueio que realmente exija o usuário.

## Fluxo

### 1. Preparar a execução

1. Identificar a pasta da feature e o repositório-alvo indicado pelos documentos.
2. Validar os três gates de aprovação e a correspondência entre `tasks.md` e os
   arquivos individuais.
3. Inspecionar branch, worktree e alterações existentes do repositório-alvo.
4. Se estiver em `main`/`master`, sincronizar com o remoto conforme `AGENTS.md` e
   criar a branch convencional da feature.
5. Se já estiver na branch correta da feature, reutilizá-la. Se estiver em uma
   branch não relacionada ou houver alterações conflitantes, pedir orientação.

A branch criada neste passo satisfaz o gate de branch das execuções individuais;
não repetir a preparação de branch ao chamar `execute-task` dentro deste fluxo.

### 2. Executar a próxima task

Enquanto existir item pendente em `tasks.md`:

1. Selecionar a primeira task pendente cujas dependências estejam concluídas.
2. Ler integralmente o arquivo da task e carregar as skills nele indicadas.
3. Para implementação comum, seguir `execute-task`, exceto pela criação de branch
   já realizada. Para tarefas especializadas, seguir a skill declarada no arquivo,
   como `execute-qa`, `execute-bugfix` ou `execute-review`.
4. Implementar todo o escopo da task.
5. Rodar primeiro as verificações direcionadas e depois as verificações exigidas
   pela própria task.
6. Acionar `task-reviewer`. Se o cliente não disponibilizar subagentes, executar
   as mesmas instruções de review no contexto principal.
7. Corrigir todos os achados acionáveis e repetir review e verificações até passar.
8. Marcar a task como concluída em `tasks.md` somente depois da validação.
9. Revisar o diff, adicionar apenas os arquivos pertencentes à task e criar um
   commit local em Conventional Commits. Não usar `git add .`.
10. Continuar imediatamente para a próxima task.

Não criar commit vazio quando uma task exclusivamente documental ou de validação
não produzir mudança rastreável no repositório-alvo.

### 3. Autocorreção e bloqueios

Ao encontrar falha:

1. Diagnosticar a causa com evidência concreta.
2. Aplicar uma correção e repetir a verificação relevante.
3. Tentar abordagens distintas antes de solicitar intervenção.

Solicitar o usuário somente quando:

- três tentativas concretas não resolverem o mesmo bloqueio;
- faltar decisão funcional, credencial ou dependência externa indispensável;
- houver conflito com mudanças preexistentes;
- uma regra de alto risco do `AGENTS.md` exigir confirmação explícita.

Antes de pedir ajuda, informar a task, o erro, o que foi tentado e a decisão
necessária. Não marcar nem commitar a task bloqueada.

### 4. Finalizar

Quando não houver tasks pendentes:

1. Confirmar que QA, bugfix aplicável e review final foram executados conforme a
   lista aprovada.
2. Rodar a verificação final prevista na TechSpec ou no `AGENTS.md`.
3. Mostrar a branch, os commits criados, os testes executados e eventuais passos
   manuais restantes.
4. Encerrar sem push e sem criar PR.

Quando o cliente oferecer objetivo persistente e o usuário tiver pedido execução
até o fim, usar esse mecanismo para sustentar o trabalho entre compactações. A
skill deve continuar funcionando mesmo quando esse recurso não estiver disponível.
