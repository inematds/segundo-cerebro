# Passo a passo prático — Segundo Cérebro pro Claude Code (Graphify + Obsidian)

Guia de mão na massa para **implementar e usar** o stack do vídeo *"Graphify + Obsidian + Claude Code = CHEAT CODE"*. Tudo aqui é **copy-run**: você cola, roda e confere. Acompanha o curso (Trilha 2), mas funciona sozinho.

> **Honestidade técnica.** No vídeo o autor chama a ferramenta de "Graphify/Graphifi" e mostra um flag `graphify --obsidian`. A ferramenta real é o pacote **`graphifyy`** (CLI Python, dois "y", de Safi Shamsi — também existe uma versão TypeScript `@sentropic/graphify`). As duas instalam a skill `/graphify` no Claude Code. Onde houver dúvida de versão, rode `graphify --help` e veja o README do repositório. Os números do vídeo (145 docs → 591 nós, 685 conexões, 67 comunidades) são de **uma execução específica** do autor, não constantes da ferramenta.

---

## 0. Pré-requisitos (5 min)

| Precisa | Como conferir / obter |
|---|---|
| **Python 3.10+** | `python --version` (ou `python3 --version`) |
| **uv** (instalador recomendado) | `curl -LsSf https://astral.sh/uv/install.sh \| sh` — ou use `pipx`/`pip` |
| **Claude Code** | já instalado e funcionando no terminal |
| **Obsidian** | baixe grátis em https://obsidian.md (desktop) |

> **Node.js não é necessário** (o Graphify é Python). **API key:** rodando via `/graphify` **dentro do Claude Code você NÃO precisa de chave** — a sessão fornece o modelo. Só o uso *headless* (terminal puro sobre documentos) pede `ANTHROPIC_API_KEY`. Para código puro, a extração é via AST (tree-sitter) e **não usa chave**.

---

## 1. Instalar o Graphify e registrar a skill (5 min)

**Objetivo:** ter o comando `graphify` e a skill `/graphify` disponível no Claude Code.

```bash
# 1) instalar a CLI (recomendado: uv)
uv tool install graphifyy
#    alternativas:  pipx install graphifyy   |   pip install graphifyy

# se "graphify" não for encontrado depois:
uv tool update-shell      # e reabra o terminal

# 2) registrar a skill no Claude Code (escreve ~/.claude/skills/graphify/SKILL.md)
graphify install
#    versão por-projeto (fica commitável no repo):
#    graphify install --project
```

**Como verificar:**
```bash
graphify --help                         # deve listar os subcomandos (extract, update, query, ...)
ls ~/.claude/skills/graphify/SKILL.md   # deve existir
```

---

## 2. Instalar e abrir o Obsidian (3 min)

**Objetivo:** ter o app pronto para receber o vault.

1. Baixe e instale o Obsidian de https://obsidian.md.
2. Abra. Não precisa criar nada ainda — vamos **apontar** o Obsidian para a pasta gerada no Passo 6.

---

## 3. Escolher e preparar a fonte (10 min)

**Objetivo:** ter uma pasta com o conteúdo que vira grafo. Pode ser **código** (repositório) ou **documentos** (PDF, markdown, etc.). No exemplo do vídeo, são os documentos oficiais do Claude Code.

**Opção A — peça ao Claude Code para baixar (prompt copy-run):**
```text
Baixe a documentação oficial do Claude Code para uma pasta local chamada
./claude-code-docs (um arquivo por página, em markdown). Liste quantos arquivos baixou.
```

**Opção B — já tenho os arquivos:** junte tudo numa pasta dedicada e limpa, ex.:
```bash
mkdir -p ~/projetos/segundo-cerebro/claude-code-docs
# copie seus .md/.pdf para dentro
```

**Dica:** comece com um subconjunto (uma dúzia de arquivos) para a primeira rodada ser rápida e barata. Tire binários, builds e duplicatas — menos ruído, grafo melhor.

**Como verificar:**
```bash
ls ~/projetos/segundo-cerebro/claude-code-docs | wc -l   # quantos arquivos no corpus
```

---

## 4. Rodar o Graphify e ler o grafo (15 min)

**Objetivo:** gerar o grafo de conhecimento e saber ler o que saiu.

**Dentro do Claude Code (skill — recomendado):**
```text
/graphify ./claude-code-docs
```
Ou por linguagem natural: *"rode o Graphify na pasta ./claude-code-docs e me dê um resumo do grafo"*.

**No terminal puro (headless — equivalente):**
```bash
graphify extract ./claude-code-docs
```

**O que sai (na pasta `graphify-out/`):**

| Arquivo | O que é | O que fazer |
|---|---|---|
| `graph.json` | O grafo completo (fonte de verdade) | não edite à mão |
| `graph.html` | Visualização interativa | **abra no navegador** |
| `GRAPH_REPORT.md` | Auditoria: **god nodes** + **perguntas sugeridas** | **leia primeiro** |
| `cache/` | Cache por arquivo | ignore (acelera re-rodadas) |

**Explorar pelo terminal (opcional):**
```bash
graphify explain "Context Window"          # explica um nó em linguagem simples
graphify path "Hooks" "Subagents"          # caminho mais curto entre dois nós
graphify query "como funcionam os hooks?"   # pergunta direto ao grafo
```

**Re-rodar barato quando a fonte muda:**
```bash
graphify update ./claude-code-docs   # só reprocessa o que mudou
# ou, na skill:  /graphify ./claude-code-docs --update   (ou --watch para auto)
```

**Como verificar:** abra `graphify-out/graph.html` no navegador e veja nós, conexões e comunidades; abra `graphify-out/GRAPH_REPORT.md` e identifique 2-3 god nodes.

---

## 5. Gerar o vault Obsidian (10 min)

**Objetivo:** transformar o grafo num vault de markdown — **um arquivo por nó**, com `[[wikilinks]]`, mais um `graph.canvas` com as comunidades agrupadas.

> O flag `--obsidian` **só existe na skill** (dentro do Claude Code), não no `graphify extract` headless.

**Dentro do Claude Code:**
```text
/graphify ./claude-code-docs --obsidian --obsidian-dir ~/vault/graphify/claude-code
```
`--obsidian-dir <pasta>` define onde o vault nasce. Se omitir, a skill cria um diretório próprio (uma "quarentena" — ver Passo 7).

**Alternativa — modo wiki** (artigos estilo Wikipédia por comunidade, com um `index.md`; ótimo para o agente navegar lendo):
```text
/graphify ./claude-code-docs --wiki
```

**Como verificar:**
```bash
ls ~/vault/graphify/claude-code | head        # vários .md (um por nó)
ls ~/vault/graphify/claude-code/*.canvas      # o graph.canvas
```

> **Importante:** o export é **regenerado do zero** a cada execução. Não edite as notas geradas esperando que sobrevivam — suas anotações próprias ficam em pastas separadas (ver Passo 8).

---

## 6. Abrir no Obsidian e conectar as fontes (10 min)

**Objetivo:** navegar o vault e ligar cada conceito ao documento de origem.

1. **Apontar o Obsidian para a pasta:** no Obsidian, canto inferior esquerdo → **Manage vaults** → **Open folder as vault** → escolha `~/vault/graphify/claude-code`.
2. **Navegar:** abra uma nota-nó e siga os `[[backlinks]]` para os conceitos ligados. Esse é o segundo cérebro em ação.
3. **Ver as comunidades:** abra o arquivo `graph.canvas` — as comunidades aparecem como grupos nomeados.
4. **Ligar às fontes (prompt copy-run no Claude Code):**
```text
Traga os documentos-fonte para dentro do vault em ~/vault/graphify/claude-code e
ligue cada nó à sua origem (um link "documento-fonte" em cada nota), para eu poder
ir do conceito direto ao texto completo.
```

> A **graph view** do Obsidian é só um desenho dos links entre as notas markdown — é um **espelho**, não o grafo original do Graphify. Sabendo disso, a expectativa fica certa.

---

## 7. As 4 estratégias de integração (decida quanto entra no seu cérebro principal)

O export pode jogar **centenas** de arquivos no seu vault. Escolha o nível de integração:

| # | Estratégia | O que é | Quando usar |
|---|---|---|---|
| 1 | **Vault standalone** | Mantém o vault gerado isolado, como um vault próprio | Você só quer o conhecimento dentro do ecossistema Obsidian |
| 2 | **Subpasta-quarentena** | Tudo numa subpasta do vault principal (ex.: `graph-imports/claude-code-docs`) que dá pra **apagar inteira** | Trazer pro contexto com saída fácil — **recomendado para começar** |
| 3 | **Importação seletiva (harvest)** | Trazer só as notas relevantes (ex.: as ~100 sobre subagents) e ignorar o resto | Você não quer despejar 600 arquivos |
| 4 | **Redistribuição** | O Claude Code espalha cada nota na subpasta que faz mais sentido | Máxima coerência — mas **mais difícil de desfazer** |

**Mover pro vault principal (prompt copy-run, estratégia 2):**
```text
Mova a estrutura do vault em ~/vault/graphify/claude-code para dentro do meu vault
principal em <caminho-do-seu-vault>, numa subpasta própria chamada
graph-imports/claude-code-docs, sem misturar com o resto.
```

**Importação seletiva (prompt copy-run, estratégia 3):**
```text
Olhe o vault gerado em ~/vault/graphify/claude-code e traga para
graph-imports/claude-code-docs apenas as notas relacionadas a <tema, ex.: subagents>.
Ignore o resto. Liste o que trouxe.
```

**Árvore de decisão rápida:**
- É um **code base** para explorar pontualmente? → **pare no Graphify** (graph.html + `graphify query`).
- Quer no Obsidian, mas **isolado**? → **standalone**.
- Quer integrar **com segurança**? → **quarentena** (apaga numa pasta só).
- Quer **curar**? → **harvest**.
- Quer **coerência total** e topa o risco? → **redistribuição**.

---

## 8. Usar no dia a dia (o "como usar")

**Perguntar usando o mapa (prompt copy-run):**
```text
Use o vault em graph-imports/claude-code-docs para me explicar <tema> e os conceitos
ligados a ele. Comece pelos nós mais conectados (god nodes), siga os [[links]] para os
vizinhos e cite as notas que usou.
```

**Servir o grafo via MCP (avançado, opcional):**
```bash
graphify --mcp        # sobe um servidor MCP com tools query_graph, get_node, get_neighbors, shortest_path
```
Assim o agente consulta o grafo programaticamente (resources `graphify://report`, `graphify://god-nodes`, `graphify://audit`).

**Manter o vault vivo:**
- Fonte mudou? `graphify update ./claude-code-docs` e re-exporte o `--obsidian`.
- Mantenha **suas** anotações separadas das notas geradas (que são regeneradas do zero).
- Versione: `git add graphify-out/` (ou o vault) para reproduzir e fazer backup.

---

## 9. Solução de problemas

| Sintoma | Causa provável | Fix |
|---|---|---|
| `graphify: command not found` | binário fora do PATH | `uv tool update-shell` e reabra o terminal |
| Erro de **API key** no terminal | uso headless sobre docs | `export ANTHROPIC_API_KEY=<sua-chave>` — **não** é preciso dentro do `/graphify` |
| `--obsidian` "não existe" | você usou o `graphify extract` headless | use o flag **na skill**: `/graphify ./docs --obsidian ...` |
| Grafo raso ou barulhento | corpus sujo / superficial | limpe a fonte, ou use `/graphify ./docs --mode deep` |
| Minhas edições sumiram do vault | export é **regenerado** | mantenha suas notas em pasta separada das geradas |
| `graphify` não acha Python | Python < 3.10 | instale 3.10+ e reinstale com `uv tool install graphifyy` |

---

## 10. Checklist final

- [ ] `graphify --help` responde e `~/.claude/skills/graphify/SKILL.md` existe
- [ ] Obsidian instalado
- [ ] Pasta-fonte pronta e contada
- [ ] `graphify-out/graph.html` abre e mostra nós/comunidades
- [ ] `graphify-out/GRAPH_REPORT.md` lido (achei 2-3 god nodes)
- [ ] Vault gerado com `--obsidian` (vários `.md` + `graph.canvas`)
- [ ] Vault aberto no Obsidian (Open folder as vault)
- [ ] Fontes ligadas aos nós
- [ ] Estratégia de integração escolhida (recomendado: quarentena)
- [ ] Fiz o Claude Code responder usando o vault

---

*Material educativo INEMA · acompanha o curso "Segundo Cérebro pro Claude Code" · https://inema.club*
