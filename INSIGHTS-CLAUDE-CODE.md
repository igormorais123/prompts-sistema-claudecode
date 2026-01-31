# INSIGHTS-CLAUDE-CODE.md

Documento de referencia extraido da analise dos system prompts oficiais do Claude Code
(v1 Sonnet 4, v2 Sonnet 4.5, Prompt Base Sonnet 4.5, Tools.json).

Dividido em duas partes:
- **Parte A**: Instrucoes operacionais para CLAUDE.md (copiar para qualquer projeto)
- **Parte B**: Dicionario de ferramentas + referencia tecnica

---

# PARTE A - Instrucoes Operacionais para CLAUDE.md

> Copie o bloco abaixo para o CLAUDE.md de qualquer projeto para otimizar a operacao.

---

## Regras de Operacao

### Comunicacao
- Responder em portugues brasileiro, termos tecnicos em ingles
- Respostas curtas (< 4 linhas) para coisas simples; detalhadas para tarefas complexas
- SEM preamble ("Vou fazer...", "Aqui esta...") ou postamble ("Espero ter ajudado...")
- SEM afirmacoes vazias ("Certamente!", "Claro!", "Absolutamente!")
- SEM emojis a menos que eu peca
- Quando recusar algo, nao explicar por que - oferecer alternativa
- Explicar comandos bash nao-triviais antes de executar

### Workflow Padrao
1. SEMPRE ler codigo antes de propor mudancas (Read primeiro)
2. Explorar codebase antes de grandes modificacoes (usar Task/Explore)
3. Para tarefas com 3+ passos: usar task tracking (TodoWrite ou TaskCreate)
4. Apos completar: rodar lint/typecheck se comandos disponiveis
5. NUNCA commitar sem pedido explicito
6. NUNCA criar arquivos .md ou README sem pedido explicito
7. NUNCA adicionar comentarios ao codigo sem pedido explicito

### Gestao de Contexto (INTEIA)
- Monitorar uso de tokens proativamente
- /clear entre tarefas distintas
- /clear apos 2 falhas consecutivas (reset > correcao)
- Manter contexto < 40% para tarefas complexas
- Usar subagentes para investigacoes em > 10 arquivos

### Anti-Over-Engineering
- Implementar APENAS o que foi pedido
- Nao adicionar features extras
- Nao criar abstracoes prematuras
- Nao refatorar codigo adjacente
- 3 linhas similares > abstracoes prematura

### Convencoes de Codigo
- Seguir estilo do arquivo existente (indentacao, naming, framework)
- Verificar bibliotecas no package.json/cargo.toml ANTES de usar
- Olhar componentes vizinhos para padroes
- NUNCA introduzir vulnerabilidades OWASP top 10
- NUNCA commitar secrets (.env, credentials.json, keys)

### Git
- Formato de commit: tipo(escopo): descricao (em ingles)
- Exemplos: `feat(auth): add JWT login`, `fix(api): handle null response`
- SEMPRE usar HEREDOC para mensagens de commit
- NUNCA force push, reset --hard, skip hooks sem pedido explicito
- NUNCA usar flags interativas (-i)
- Verificar authorship antes de amend

### Plan Mode
- Usar para: implementacoes novas, mudancas arquiteturais, features complexas
- NAO usar para: pesquisa, exploracao, leitura de arquivos
- Fluxo: Explorar (Glob/Grep/Read) -> Planejar -> Apresentar -> ExitPlanMode -> Implementar

### Verificacao Obrigatoria
- Definir criterios de sucesso testaveis
- Preferir: testes > linter > screenshots > verificacao manual
- NUNCA marcar tarefa como completed se ha erros nao resolvidos

---

# PARTE B - Dicionario de Ferramentas

## Ferramentas de Arquivo

### Read
- **O que faz**: Le conteudo de arquivo (texto, imagem, PDF, .ipynb)
- **Quando usar**: SEMPRE antes de editar; para ver codigo; para ler screenshots
- **Parametros**: `file_path` (obrigatorio, absoluto), `offset`, `limit`
- **Notas**: Retorna ate 2000 linhas, formato cat -n. Linhas > 2000 chars sao truncadas.
- **Substitui**: cat, head, tail

### Edit
- **O que faz**: Substituicao exata de string em arquivo
- **Quando usar**: Mudancas pontuais em arquivos existentes
- **Parametros**: `file_path`, `old_string` (deve ser UNICO), `new_string`, `replace_all`
- **Notas**: EXIGE Read previo. Falha se old_string nao for unica. Preservar indentacao.
- **Substitui**: sed, awk

### MultiEdit
- **O que faz**: Multiplas substituicoes no mesmo arquivo em uma operacao
- **Quando usar**: Varias mudancas no mesmo arquivo (mais eficiente que Edit repetido)
- **Parametros**: `file_path`, `edits[]` (array de {old_string, new_string, replace_all})
- **Notas**: Edits aplicados em sequencia. Atomico: todos falham se um falha.

### Write
- **O que faz**: Cria ou sobrescreve arquivo inteiro
- **Quando usar**: APENAS para criar arquivos NOVOS ou reescrever completamente
- **Parametros**: `file_path` (absoluto), `content`
- **Notas**: EXIGE Read previo para arquivos existentes. Preferir Edit quando possivel.
- **Substitui**: echo >, cat <<EOF

### NotebookEdit
- **O que faz**: Edita celulas de Jupyter Notebook
- **Quando usar**: Modificar .ipynb
- **Parametros**: `notebook_path`, `cell_id`, `new_source`, `cell_type`, `edit_mode`
- **Notas**: cell_number e 0-indexed. edit_mode: replace, insert, delete.

---

## Ferramentas de Busca

### Glob
- **O que faz**: Busca arquivos por padrao de nome (glob)
- **Quando usar**: Encontrar arquivos por nome/extensao
- **Parametros**: `pattern` (ex: "**/*.ts"), `path` (diretorio opcional)
- **Notas**: Retorna caminhos ordenados por data de modificacao.
- **Substitui**: find, ls

### Grep
- **O que faz**: Busca conteudo dentro de arquivos (baseado em ripgrep)
- **Quando usar**: Encontrar codigo, strings, padroes dentro de arquivos
- **Parametros**: `pattern` (regex), `path`, `glob` (filtro de arquivo), `output_mode`
- **Output modes**: `files_with_matches` (padrao), `content`, `count`
- **Extras**: `-A`, `-B`, `-C` (contexto), `-i` (case insensitive), `-n` (line numbers)
- **Notas**: Suporta regex completo. Para multiline: `multiline: true`
- **Substitui**: grep, rg (via Bash)

---

## Ferramentas de Agente

### Task
- **O que faz**: Lanca subagente autonomo para tarefas complexas
- **Quando usar**: Buscas abertas, investigacoes multi-arquivo, tarefas multi-passo
- **Parametros**: `description` (3-5 palavras), `prompt`, `subagent_type`
- **Tipos de agente disponíveis**:

| Tipo | Uso | Ferramentas |
|------|-----|-------------|
| `general-purpose` | Pesquisa complexa, busca de codigo | Todas |
| `Explore` | Exploracao rapida de codebase | Todas exceto Task, Edit, Write |
| `Plan` | Planejamento de implementacao | Todas exceto Task, Edit, Write |
| `Bash` | Execucao de comandos, git | Bash |
| `statusline-setup` | Configurar status line | Read, Edit |
| `output-style-setup` | Criar estilo de output | Read, Write, Edit, Glob, Grep |

- **Notas**:
  - Cada invocacao e STATELESS (sem contexto previo)
  - Lanciar multiplos em paralelo quando possivel
  - Resultado nao e visivel ao usuario - resuma ao responder
  - Prefira Explore para questoes abertas sobre o codebase

### EnterPlanMode
- **O que faz**: Transiciona para modo planejamento
- **Quando usar**: Antes de implementacoes nao-triviais
- **Notas**: Requer aprovacao do usuario

### ExitPlanMode
- **O que faz**: Sai do modo planejamento e solicita aprovacao
- **Quando usar**: Apos finalizar plano de implementacao
- **Notas**: NAO usar para pesquisa/exploracao. Sinaliza que o plano esta pronto.

---

## Ferramentas de Tarefa

### TaskCreate / TodoWrite
- **O que faz**: Cria lista de tarefas estruturada
- **Quando usar**: Tarefas com 3+ passos; features multiplas; planejamento
- **Estados**: `pending` -> `in_progress` -> `completed`
- **Regras**:
  - Maximo 1 tarefa in_progress por vez
  - Marcar completed IMEDIATAMENTE ao terminar
  - NUNCA marcar completed com erros pendentes

### TaskUpdate
- **O que faz**: Atualiza status de tarefa
- **Quando usar**: Ao iniciar, completar, ou encontrar bloqueio

### TaskList
- **O que faz**: Lista todas as tarefas
- **Quando usar**: Ver progresso, encontrar proxima tarefa

---

## Ferramentas Web

### WebFetch
- **O que faz**: Busca e processa conteudo de URL
- **Quando usar**: Ler documentacao web, analisar paginas
- **Parametros**: `url`, `prompt` (o que extrair)
- **Notas**: Cache de 15 min. Se redirect para outro host: fazer novo request.

### WebSearch
- **O que faz**: Busca na web
- **Quando usar**: Informacoes alem do knowledge cutoff, dados em tempo real
- **Parametros**: `query`, `allowed_domains`, `blocked_domains`

---

## Ferramentas de Terminal

### Bash
- **O que faz**: Executa comando no terminal
- **Quando usar**: git, npm, docker, comandos do sistema
- **Parametros**: `command`, `timeout` (max 600000ms), `description`, `run_in_background`
- **NUNCA usar para**: ler/editar/buscar arquivos (usar ferramentas dedicadas)
- **Notas**:
  - Sempre colocar aspas em caminhos com espacos
  - Multiplos comandos: `&&` (sequencial) ou `;` (independente)
  - Preferir caminhos absolutos, evitar cd
  - Para background: usar `run_in_background: true`

### BashOutput / TaskOutput
- **O que faz**: Recupera output de shell em background
- **Quando usar**: Monitorar comandos longos rodando em background

### KillShell / TaskStop
- **O que faz**: Mata shell em background
- **Quando usar**: Terminar processo longo desnecessario

---

## Ferramentas MCP (se configuradas)

### Servidores MCP Comuns
- **github**: Operacoes GitHub (repos, issues, PRs, branches)
- **filesystem**: Operacoes de arquivo com controle de diretorio
- **memory**: Knowledge graph persistente
- **puppeteer/playwright**: Automacao de browser
- **context7**: Documentacao atualizada de bibliotecas
- **sequential-thinking**: Raciocinio estruturado passo-a-passo

### Como Identificar
- Ferramentas MCP comecam com `mcp__` (ex: `mcp__github__create_issue`)
- Preferir ferramentas MCP sobre WebFetch quando disponiveis

---

## Ferramentas de Interacao

### AskUserQuestion
- **O que faz**: Faz perguntas ao usuario durante execucao
- **Quando usar**: Clarificar requisitos, validar premissas, escolher abordagem
- **Notas**: Maximo 4 perguntas, 2-4 opcoes cada

---

# PARTE C - Padroes de Uso Otimizado

## Paralelismo (SEMPRE que possivel)

```
Independentes -> Paralelo:
  Read(A) + Read(B) + Read(C)
  git status + git diff + git log
  Grep(padrao1) + Grep(padrao2)

Dependentes -> Sequencial:
  Read(arquivo) -> Edit(arquivo)
  git add -> git commit -> git status
  Glob(buscar) -> Read(resultado)
```

## Decisao de Ferramenta por Cenario

| Cenario | Ferramenta | NAO usar |
|---------|-----------|----------|
| "Onde esta a funcao X?" | Grep pattern + Glob | bash grep |
| "O que faz esse arquivo?" | Read | bash cat |
| "Mude X para Y" | Edit (Read primeiro) | bash sed |
| "Crie arquivo novo" | Write | bash echo > |
| "Como funciona o codebase?" | Task/Explore | multiplos Grep manuais |
| "Implemente feature X" | EnterPlanMode -> implementar | comecar direto |
| "Commite as mudancas" | Bash (git add && commit) | - |
| "Crie PR" | Bash (gh pr create) | - |
| "Busque na web" | WebSearch/WebFetch | - |
| "Monitore o build" | Bash (background) + TaskOutput | - |

## Templates de Prompt Eficientes

### Para Mudancas Pontuais
```
Mude [X] para [Y] no arquivo [Z]. Apenas isso.
```

### Para Exploracao
```
Explore o codebase e explique como [funcionalidade] funciona.
Liste os arquivos relevantes.
```

### Para Implementacao
```
Implemente [feature]. Requisitos:
1. [requisito testavel 1]
2. [requisito testavel 2]
Use Plan Mode. Siga convencoes existentes.
```

### Para Debug
```
O erro [X] ocorre quando [Y].
Investigue a causa raiz e proponha fix.
NAO implemente ainda, apenas diagnostique.
```

### Para Commit
```
Commite as mudancas. Formato: tipo(escopo): descricao em ingles.
```

---

# PARTE D - Anatomia do Sistema (Como Claude Code Funciona)

## Arquitetura Interna
1. **System Prompt** define: personalidade, ferramentas, regras, tom
2. **CLAUDE.md** (projeto/global) adiciona: instrucoes customizadas do usuario
3. **Tools** sao chamadas via XML pelo modelo, executadas pelo runtime
4. **Subagentes (Task)** sao processos independentes sem contexto compartilhado
5. **Hooks** sao shell commands que executam em resposta a tool calls
6. **system-reminder tags** sao inseridas automaticamente pelo sistema, nao pelo usuario

## Fluxo de uma Sessao
```
[Usuario] -> mensagem
  -> [System Prompt + CLAUDE.md + Contexto] processados
    -> [Claude] decide ferramenta(s)
      -> [Runtime] executa ferramenta(s)
        -> [Claude] analisa resultado
          -> [Claude] responde ou chama mais ferramentas
```

## Evolucao das Versoes Documentadas
| Versao | Modelo | Data | Novidades |
|--------|--------|------|-----------|
| v1 | Sonnet 4 | 2025-08 | Base: Edit, Read, Write, Bash, Glob, Grep, Task, TodoWrite |
| v2 | Sonnet 4.5 | 2025-09 | +MultiEdit, +KillBash, Agent SDK, docs.claude.com |
| Atual | Opus 4.5 | 2026-01 | +MCP, +Playwright, +Puppeteer, +TaskCreate/Update/List, +EnterPlanMode, +AskUserQuestion, +Explore/Plan/Bash agents |

## Modelo Base (Sonnet 4.5 Chat) vs Claude Code
| Aspecto | Chat | Claude Code |
|---------|------|-------------|
| Foco | Conversacao natural | Engenharia de software |
| Output | Artifacts (React, SVG, HTML) | Arquivos locais |
| Busca | web_search com citations | WebSearch + WebFetch |
| Memoria | Nenhuma entre sessoes | CLAUDE.md + knowledge graph MCP |
| Seguranca | Copyright, harmful content | Filesystem, git, secrets |
| Ferramentas | ~3 (artifacts, search, fetch) | 15+ nativas + MCP ilimitado |

---

*Gerado da analise dos system prompts oficiais do Claude Code.*
*Versoes analisadas: v1 (Sonnet 4, ago/2025), v2 (Sonnet 4.5, set/2025), base Sonnet 4.5, Tools.json*
*Runtime atual: Opus 4.5, jan/2026*
