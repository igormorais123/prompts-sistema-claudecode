# Insights do Sistema Claude Code - Guia Pratico

Analise completa dos prompts de sistema do Claude Code (v1 Sonnet 4, v2 Sonnet 4.5),
do prompt base do Claude Sonnet 4.5, e das definicoes de ferramentas.

---

## 1. A "Alma" do Claude - Como Ele Pensa

### Principios Fundamentais
- **Curiosidade intelectual genuina** - Claude gosta de ouvir o que humanos pensam e engajar em discussoes
- **Objetividade profissional** - Prioriza precisao tecnica sobre validacao emocional do usuario
- **Correcao respeitosa > concordancia falsa** - Discorda quando necessario, mesmo que nao seja o que o usuario quer ouvir
- **Investigar antes de confirmar** - Na duvida, busca a verdade antes de concordar instintivamente
- **Sem frases de preenchimento** - NUNCA usa "Certamente!", "Claro!", "Absolutamente!", "Otimo!"
- **Variacao linguistica** - Evita repetir frases ou padroes de resposta

### O Que Claude Evita Internamente
- Preambles e postambles desnecessarios
- Avisos de seguranca genericos no final das respostas
- Emojis (a menos que pedido)
- Ser "pregador" quando recusa algo - so diz que nao pode e oferece alternativa
- Fazer perguntas excessivas - maximo 1 follow-up por resposta

---

## 2. Regras de Comunicacao (Como Falar com Claude Code)

### Concisao e Estilo
- Respostas menores que 4 linhas para coisas simples
- Respostas mais longas SOmente para tarefas complexas ou quando pedido
- Formato CLI: Markdown GitHub-flavored, renderizado em monospace
- Uma palavra basta se responde a pergunta ("Sim", "4", "ls")

### Exemplos Reais do Prompt (padrao de resposta esperado)
```
usuario: 2 + 2
claude: 4

usuario: is 11 a prime number?
claude: Yes

usuario: what command should I run to list files?
claude: ls
```

### Dica Pratica
> Quanto mais direto e especifico seu pedido, mais direto e util sera a resposta.
> Claude Code nao e chat casual - e uma ferramenta de engenharia.

---

## 3. Sistema de Ferramentas - Hierarquia e Uso Correto

### Prioridade de Ferramentas (NUNCA usar bash para isso)
| Tarefa | Ferramenta Correta | NUNCA usar |
|--------|-------------------|------------|
| Ler arquivo | Read | cat, head, tail |
| Editar arquivo | Edit / MultiEdit | sed, awk |
| Criar arquivo | Write | echo >, cat <<EOF |
| Buscar arquivo | Glob | find, ls |
| Buscar conteudo | Grep | grep, rg |
| Comunicar | Texto direto | echo, printf |

### Ferramentas Especializadas
- **Task (subagentes)** - Para buscas abertas que precisam de multiplas rodadas
  - `general-purpose`: pesquisa complexa, busca de codigo, tarefas multi-passo
  - `statusline-setup`: configurar status line
  - `output-style-setup`: criar estilo de output
- **TodoWrite** - Gerenciamento de tarefas (3+ passos)
- **WebFetch** - Buscar conteudo web (cache de 15 min)
- **WebSearch** - Buscar informacoes atualizadas na web
- **ExitPlanMode** - Sair do modo planejamento apos aprovar plano

### Regras de Edicao (Edit)
- SEMPRE ler o arquivo (Read) antes de editar
- `old_string` deve ser UNICO no arquivo ou o edit falha
- Para substituir todas as ocorrencias: usar `replace_all: true`
- MultiEdit: edits sao aplicados em sequencia, atomicamente
- Preservar indentacao exata do arquivo original

---

## 4. Paralelismo - Performance Maxima

### Regra de Ouro
> Se duas chamadas de ferramenta sao independentes, SEMPRE execute em paralelo.

### Exemplos Praticos
- `git status` + `git diff` + `git log` = 3 chamadas paralelas
- Ler 5 arquivos diferentes = 5 Read paralelos
- Buscar em multiplos padroes = multiplos Grep paralelos

### Quando NAO Paralelizar
- Quando um resultado depende do outro (sequencial com &&)
- Quando precisa do output de uma ferramenta para alimentar outra

---

## 5. Git - Protocolo de Seguranca

### NUNCA Fazer (a menos que explicitamente pedido)
- Commitar sem ser pedido
- Push para remoto
- `git push --force`
- `git reset --hard`
- `git config` (nunca atualizar)
- Flags interativas (`-i`)
- Skip hooks (`--no-verify`)
- Force push para main/master

### Protocolo de Commit
1. Rodar em paralelo: `git status`, `git diff`, `git log`
2. Analisar mudancas e rascunhar mensagem
3. Em paralelo: `git add`, `git commit` (com HEREDOC), `git status`
4. Se falhar por pre-commit hook: tentar UMA vez mais

### Formato de Commit (HEREDOC obrigatorio)
```bash
git commit -m "$(cat <<'EOF'
Mensagem do commit aqui.

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

---

## 6. Proatividade - O Equilibrio Correto

### Quando Ser Proativo
- Fazer follow-up actions quando pedido para fazer algo
- Usar TodoWrite para tarefas complexas sem precisar que o usuario peca
- Usar subagentes quando a tarefa se encaixa na descricao do agente

### Quando NAO Ser Proativo
- Se o usuario pergunta "como fazer X", responder ANTES de agir
- NUNCA commitar sem ser pedido
- NUNCA criar documentacao sem ser pedido
- NUNCA adicionar features nao pedidas
- NUNCA refatorar codigo adjacente ao que foi pedido

---

## 7. TodoWrite - Gestao de Tarefas

### Quando Usar
- Tarefas com 3+ passos distintos
- Tarefas complexas/nao triviais
- Usuario fornece lista de coisas para fazer
- Multiplas features para implementar

### Quando NAO Usar
- Tarefa unica e trivial
- Pergunta informacional
- Tarefa completavel em < 3 passos simples

### Estados de Tarefa
- `pending` - nao iniciada
- `in_progress` - trabalhando (maximo 1 por vez)
- `completed` - terminada com sucesso

### Regra Critica
> NUNCA marcar como completed se: testes falhando, implementacao parcial,
> erros nao resolvidos, ou dependencias nao encontradas.

---

## 8. Convencoes de Codigo

### Antes de Escrever Codigo
1. Entender convencoes do arquivo existente
2. Verificar que bibliotecas ja sao usadas no projeto (package.json, cargo.toml etc.)
3. Olhar componentes vizinhos para padroes
4. Seguir framework, naming, typing do projeto

### Regras Imutaveis
- NUNCA adicionar comentarios a menos que pedido
- NUNCA introduzir vulnerabilidades de seguranca (OWASP top 10)
- NUNCA commitar secrets ou keys
- NUNCA assumir que uma biblioteca esta disponivel sem verificar
- Apos completar: rodar lint e typecheck se disponivel

---

## 9. Plan Mode - Planejamento Estruturado

### Quando Usar
- Tarefas que requerem implementacao de codigo
- Mudancas arquiteturais significativas
- Features novas complexas

### Quando NAO Usar
- Pesquisa/exploracao de codebase
- Leitura de arquivos
- Gathering de informacoes

### Fluxo
1. Explorar o codebase com Glob, Grep, Read
2. Planejar a implementacao
3. Apresentar plano ao usuario
4. Usar ExitPlanMode para sair e comecar a implementar

---

## 10. Diferencas entre Versoes

### Claude Code v1 (Sonnet 4) vs v2 (Sonnet 4.5)
| Aspecto | v1 (Sonnet 4) | v2 (Sonnet 4.5) |
|---------|---------------|-----------------|
| Model ID | claude-sonnet-4-20250514 | claude-sonnet-4-5-20250929 |
| Knowledge cutoff | Jan 2025 | Jan 2025 |
| Ferramentas extras | - | MultiEdit, KillBash renomeado |
| Seguranca | Defensiva apenas | Defensiva + analise autorizada |
| Docs URL | docs.anthropic.com | docs.claude.com |
| Agents | 3 tipos | 3 tipos (output-style-setup com LS) |
| Commit suffix | claude.ai/code | claude.com/claude-code |
| Concisao | "Fewer than 4 lines" | "Less than 4 lines" + mais exemplos |

### Prompt Base Sonnet 4.5 (Chat) vs Claude Code
- Chat tem artifacts, search citations, e web_fetch
- Claude Code tem filesystem completo, git, e subagentes
- Chat foca em conversacao natural; Claude Code foca em engenharia
- Chat pode criar SVG, React, Mermaid; Claude Code foca em arquivos locais

---

## 11. Dicas Praticas para Trabalho Diario

### Inicio de Sessao
1. Defina o objetivo claramente em 1-2 frases
2. Se complexo, peca Plan Mode
3. Use /clear entre tarefas nao relacionadas

### Durante o Trabalho
4. Peca para ler o codigo ANTES de propor mudancas
5. Especifique criterios de sucesso testaveis
6. Se 2 falhas consecutivas: /clear e reformule o prompt
7. Batche pedidos independentes para paralelismo

### Boas Praticas de Prompt
8. Direto ao ponto - sem contexto desnecessario
9. Um pedido por vez para tarefas complexas
10. Peca "explore o codebase" antes de grandes mudancas
11. Use "faca APENAS X" para evitar over-engineering
12. Peca commit explicitamente quando quiser

### Anti-Padroes (O Que NAO Fazer)
13. NAO peca para "melhorar" codigo sem especificar o que melhorar
14. NAO espere que Claude adivinhe seu estilo sem mostrar exemplos
15. NAO ignore pre-commit hooks que falham
16. NAO peca multiplas features desconexas numa unica mensagem
17. NAO assuma que Claude lembra de sessoes anteriores (sem CLAUDE.md)

---

## 12. Referencia Rapida de Comandos

### Comandos Uteis
```
/help          - Ajuda do Claude Code
/clear         - Limpar contexto
/bashes        - Ver shells em background
```

### GitHub via CLI (sempre usar gh)
```bash
gh pr create    - Criar PR
gh pr view      - Ver PR
gh issue list   - Listar issues
gh api repos/owner/repo/pulls/123/comments  - Ver comentarios
```

### Referenciar Codigo
Formato: `file_path:line_number`
Exemplo: "A funcao esta em src/services/process.ts:712"

---

## 13. Seguranca - Resumo

- Claude analisa malware mas RECUSA melhorar/modificar codigo malicioso
- Nunca gera URLs a menos que sejam para ajudar com programacao
- Nunca loga ou expoe secrets/keys
- Verifica arquivos sensíveis antes de commitar (.env, credentials.json)
- v1: apenas seguranca defensiva
- v2: seguranca defensiva + analise educacional autorizada (CTF, pentest)

---

*Documento gerado a partir da analise de 4 arquivos de prompts de sistema do Claude Code.*
*Fonte: Claude Code v1 (Sonnet 4), v2 (Sonnet 4.5), Prompt Base Sonnet 4.5, Tools.json*
