# Guia Rapido: Como Claude Code Funciona (Para Humanos)

## Como o Sistema Funciona
- Claude Code recebe um **system prompt** gigante com personalidade, regras e ferramentas
- Seu **CLAUDE.md** e lido em cima disso e tem prioridade para instrucoes de projeto
- Cada mensagem sua e processada contra tudo isso + contexto da conversa
- Ferramentas (Read, Edit, Bash, etc.) sao executadas pelo runtime, nao pelo modelo
- Subagentes (Task) sao processos separados SEM memoria da conversa atual
- Contexto e finito: conversas longas perdem informacao do inicio

## O Que Claude Code Sabe Fazer (e Voce Talvez Nao Saiba)
- Ler imagens, PDFs e Jupyter Notebooks diretamente
- Rodar comandos em background e monitorar depois
- Lancar multiplos subagentes em paralelo para pesquisas
- Buscar documentacao atualizada na web (WebSearch/WebFetch)
- Usar servidores MCP (GitHub, Playwright, Google Drive, etc.)
- Entrar em Plan Mode para planejar antes de implementar
- Fazer perguntas de clarificacao durante execucao (AskUserQuestion)

## Como Voce Deve Se Comunicar
- **Direto ao ponto.** "Mude X para Y em Z" ganha de "poderia melhorar esse codigo?"
- **Um pedido por vez** para coisas complexas. Multiplos pedidos simples podem ser juntos.
- **Especifique criterios de sucesso.** "Deve passar nos testes" e melhor que "faca funcionar"
- **Diga "apenas isso"** quando nao quiser extras. Claude tende a over-engineer sem freio.
- **Peca Plan Mode** para features novas. Evita retrabalho.
- **Peca commit explicitamente.** Claude NUNCA deve commitar sozinho.

## Erros Comuns a Evitar
- Pedir para "melhorar" sem dizer o que melhorar (gasta tokens em adivinhacao)
- Insistir apos 2 falhas (melhor: /clear e reformular)
- Conversas muito longas sem /clear (contexto degrada, respostas pioram)
- Assumir que Claude lembra sessoes anteriores (nao lembra, use CLAUDE.md)
- Colocar instrucoes longas no CLAUDE.md (cada token e lido toda sessao)

## O Que Colocar no CLAUDE.md (Maximo 20 Linhas)
Apenas informacoes que Claude NAO tem como saber sozinho:
- Stack: "Next.js 14, TypeScript, Prisma, PostgreSQL"
- Comandos: "npm run test", "npm run lint", "npm run build"
- Convencoes: "commits em ingles, formato conventional commits"
- Arquivos criticos: "config em /src/config, API em /src/api"
- Idioma: "Responder em portugues brasileiro"

## Quando Usar Cada Recurso
- **/clear** - Entre tarefas diferentes ou apos 2 falhas
- **Plan Mode** - Features novas, mudancas arquiteturais
- **Subagentes** - Exploracoes de codebase (>5 arquivos)
- **Task tracking** - Tarefas com 3+ passos
- **Background bash** - Builds, testes longos, servidores dev

## Custo-Beneficio de Tokens
- CLAUDE.md enxuto (20 linhas) = ~200 tokens/sessao = custo negligivel
- CLAUDE.md extenso (200 linhas) = ~2000 tokens/sessao = custo real sem beneficio proporcional
- Subagentes consomem tokens proprios mas preservam contexto principal
- /clear e gratuito e reseta o contexto para maximo rendimento
- Conversas >50 mensagens degradam: prefira sessoes curtas e focadas
