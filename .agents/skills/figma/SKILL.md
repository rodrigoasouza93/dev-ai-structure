---
name: figma
description: >-
  Lê designs do Figma via MCP para planejar e implementar telas da Plataforma
  Bliss (intranet). Use quando o usuário mandar um link do figma.com, pedir para
  localizar/consultar uma tela do design, decidir onde encaixar uma nova
  funcionalidade na UI existente, ou traduzir um frame em código React.

  Não use para desenhar/editar no Figma — o acesso atual é somente leitura.
---

# Figma (Plataforma Bliss)

## Visão geral

O MCP oficial do Figma já vem conectado como connector do claude.ai (ferramentas
`mcp__claude_ai_Figma__*`). Não há nada a instalar nem `.mcp.json` a configurar.
Esta skill existe para evitar as armadilhas concretas do arquivo de design da
Bliss, que é grande o bastante para estourar o contexto se consultado sem
cuidado.

## Pré-requisitos

- Connector "Figma" ativo no cliente. Se as ferramentas não aparecerem, rode
  `/mcp` no Claude Code e reconecte.
- O app **Figma Desktop** aberto ajuda: as respostas passam a incluir um bloco
  `Currently selected nodes:` com o que o usuário tem selecionado no canvas —
  atalho barato para saber de qual tela ele está falando.
- Confirme o acesso com `whoami` antes de investigar erro de permissão ou rate
  limit.

## Escrita no Figma não está disponível

O seat atual é **View** em plano **starter**. Ferramentas de escrita
(`use_figma`, `create_new_file`, `generate_diagram`, `upload_assets`) vão falhar.
Se o usuário pedir para criar ou editar design, diga isso de saída e ofereça a
alternativa: descrever o layout em texto/ASCII, ou prototipar em código React
para ele levar ao designer.

## Arquivo de referência

| | |
|---|---|
| Arquivo | **Plataforma Bliss** |
| `fileKey` | `pAj4aCaKywtx6jNcduJH5G` |
| Página principal | `1:13` — canvas **"Entrega"** (telas entregues/aprovadas) |

Todo o produto (intranet, visão corretor, visão cliente, mobile, login) mora na
mesma página `Entrega`. Frames de topo: ~1300. XML completo da página: ~6,4 MB.

## Armadilhas (todas verificadas na prática)

1. **Nunca chame `get_metadata` na página inteira esperando ler a resposta.**
   `get_metadata(fileKey, nodeId="1:13")` devolve ~7,3 M caracteres e estoura o
   limite de tokens. O harness salva o resultado em arquivo — o caminho volta na
   mensagem de erro. Isso é o fluxo *normal*, não uma falha: filtre o arquivo com
   `python3`/`rg` em vez de ler linha a linha.

2. **`get_metadata` sem `nodeId` não lista as páginas de verdade.** Nesse arquivo
   ele retorna apenas `0:1: cover`. Use direto o canvas `1:13`.

3. **Nomes de frame mentem.** O frame `8873:104` chama-se "Visão Proposta" e é um
   **kanban por responsável**, não a tela de detalhe da proposta. Sempre confirme
   com `get_screenshot` antes de concluir qualquer coisa sobre um frame.

4. **Há muitos frames homônimos.** "Cadastro na operadora", "Loader de criação",
   "Checklist pro cliente" e "Proposta já emitida" aparecem repetidos em fluxos
   diferentes (visão intranet × visão corretor). Cite sempre o node ID, nunca só
   o nome.

5. **O seat View tem cota de chamadas ao MCP.** Depois de ~6 chamadas numa
   sessão, tudo passa a responder *"You've reached the Figma MCP tool call
   limit"*. Planeje: uma passada de `get_metadata` (salva em arquivo, reusável à
   vontade sem gastar cota) e só então os screenshots que realmente importam.
   Se a cota estourar, siga pelo metadata já salvo e diga ao usuário o que não
   deu para conferir visualmente — não fique retentando.

6. **Duas gerações de produto no mesmo arquivo.** A maior parte das telas é da
   **Plataforma Bliss** (produto novo, sidebar roxa, accordion na lista). A
   **intranet atual** (React + Ant Design) tem seção própria, depois do marcador
   `3696:7013` ("Versão intranet →"). Antes de tratar um frame como spec da
   intranet, confirme de qual geração ele é — o layout da Plataforma **não**
   descreve a intranet atual.

## Fluxo recomendado

### 1. Do link à área certa

Extraia `fileKey` e `node-id` da URL (`?node-id=1-13` → `nodeId: "1:13"`).

### 2. Localize frames por nome, sem ler o XML inteiro

```bash
python3 - <<'PY'
import json, re
p = "<caminho-do-arquivo-salvo-pelo-harness>"
xml = "".join(x["text"] for x in json.load(open(p)))
for m in re.finditer(r'<(\w[\w-]*) id="([^"]+)" name="([^"]*)"', xml):
    if re.search(r"<termo>", m.group(3), re.I):
        print(m.group(2), m.group(1), m.group(3))
PY
```

Para listar só os frames de topo, ancore a indentação de dois espaços:
`^  <(\w[\w-]*) id="([^"]+)" name="([^"]*)"`.

Buscar pelo **texto** dos nós funciona bem: os layers de texto do arquivo têm o
próprio conteúdo como nome (`"Pendência na proposta"`, `"Status não mapeado"`),
então o nome do layer é um índice pesquisável do produto.

### 3. Veja antes de decidir

`get_screenshot` com `maxDimension: 1400` para telas de 1512px. A resposta traz
uma URL curta-vida — baixe com `curl -sL -o <nome>.png` no scratchpad e abra com
`Read`. Não use `enableBase64Response`: gasta muito mais contexto.

Prefira 2–4 screenshots bem escolhidos a um varredura ampla; cada imagem custa
contexto.

### 4. Só então extraia código

`get_design_context` **apenas no frame final**, nunca em container grande. Antes
de chamar, carregue a orientação da própria Figma:
`get_figma_skill(uri="skill://figma/figma-design-to-code/SKILL.md")`.

O código que ele devolve é **referência, não entrega**. Adapte aos padrões do
repo: siga `react-frontend-conventions` (React FC + Ant Design + TanStack
Query), `repo-folder-structure` e `ui-ux-pro-max`. Reaproveite componentes
existentes em vez de recriar o markup do Figma.

## Índice — telas de proposta e pendência

Levantado para o trabalho de Banco de Pendências; vale como ponto de partida.

| Node | Frame | O que é |
|---|---|---|
| `1942:2069` | Pendência identificada via automatização | Lista de propostas com accordion aberto, badge de status e timeline da pendência |
| `1942:1736` | Aba atividades | Mesmo accordion, aba Atividades com histórico e diffs de campo |
| `1942:493` | Lista de propostas | Lista sem accordion expandido |
| `1942:1407` | Status não mapeado | Estado de status sem mapeamento |
| `1:3286` | Qual é a pendência | Modal de criação manual de pendência ("Salvar e enviar" notifica o corretor) |
| `1:1094` | Abas + filtro + atalhos | Abas da lista + filtros e atalhos de status |
| `1:2974` / `614:2381` | Filtros abertos | Painel de filtros (inclui "Com pendência") |
| `8873:104` | "Visão Proposta" | Kanban por responsável (nome enganoso) |
| `300:15089` | Aba atividades: Histórico | Visualização de histórico |
| `344:16581` | Aba anexos preenchida | Aba Anexos |
| `396:8` | Aba dados do lead vazia | Aba Dados do lead |

### Seção da intranet atual (depois do marcador `3696:7013`)

É esta a geração que corresponde ao repositório `intranet`. Os nomes casam com as
abas reais de `pages/orderDetails/index.tsx` (`order`, `timeline`, `activity`,
`lead`, `responsible`):

| Node | Frame |
|---|---|
| `3696:7016` / `3696:7179` | Aba proposta ativa |
| `3696:7337` | Aba linha do tempo ativa |
| `3696:7452` | Aba Atividades |
| `3696:7582` | Aba Dados do lead |
| `3696:7693` | Aba Responsáveis |
| `3696:7927` | Aba comentários |

**Estrutura do accordion da proposta** — atenção, isto é da **Plataforma**, não da
intranet atual (o padrão a respeitar ao propor UI para a Plataforma):

- Linha da lista: Código da operadora · Operadora · Lead · Supervisor(a) ·
  Cadastrada na Sisweb · **Status** (badge colorido) · chevron.
- Ao expandir: abas **Proposta | Atividades | Anexos | Dados do lead**, conteúdo
  à esquerda e painel fixo à direita com badge de status (dropdown), card do
  plano, **Abrir proposta →**, toggle Sisweb, nº na operadora e corretor.

## Vocabulário de pendência já existente no design

Reutilize estes termos em vez de inventar novos:

- `Pendência na proposta` — badge laranja de status.
- `Pedido com pendência` — status na visão do corretor/cliente.
- `Pendência notificada ao corretor` — evento na timeline de atividades.
- `Com pendência` — opção de filtro na lista.
- `Status não mapeado` — badge vermelho (é buraco de mapeamento, não status de
  funil).
