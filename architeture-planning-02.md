# Registro de Decisões — Sessão de Grilling (Arquitetura Factory)

**Status:** Aceito
**Data:** 2026-10-08
**Método:** Entrevista estruturada ("grilling") sobre o plano consolidado na v2/v3 do `architeture-planning-01.md`

## Metodologia

As decisões abaixo saíram de uma entrevista estruturada, não de uma conversa livre. O processo:

1. O plano foi mapeado como uma **árvore de decisão**: cada escolha em aberto vira um nó, e escolhas que dependem de outra ainda não resolvida ficam bloqueadas até a dependência fechar.
2. A cada rodada, só as perguntas sem dependência pendente (a **fronteira** da árvore) eram feitas de uma vez — numeradas, cada uma com uma recomendação explícita anexada, nunca em aberto.
3. Cada resposta do usuário reabria a fronteira: decisões que dependiam dela viravam perguntáveis na rodada seguinte.
4. A sessão terminou quando a fronteira esvaziou — nenhum ramo da árvore ficou assumido silenciosamente.

Resultado: 19 decisões, em 4 rodadas, cobrindo execução, infraestrutura, escopo, segurança e interface.

## Decisões — Execução & Durabilidade

| # | Pergunta | Decisão | Racional |
|---|---|---|---|
| 1 | Motor de execução da Fase 2 | **OpenHands** | Entrega o ciclo plan→code→test→debug já pronto dentro de sandbox Docker; reconstruir isso na mão contraria o princípio de reaproveitar ferramentas existentes |
| 2 | Durabilidade do loop (sobreviver a crash/pausa longa) | **SQLite** no MVP, não Temporal | Peça isolada — troca por Temporal depois é possível sem reescrever o resto; reduz superfície de infra nova enquanto o resto do pipeline ainda está sendo validado |
| 11 | Teto de retry por ticket travado | **5 tentativas OU 30 minutos**, o que vier primeiro → pausa e notifica | Evita o OpenHands consumir tokens indefinidamente num ticket sem progresso |
| 18 | Escalonamento quando um bloco inteiro reprova repetidamente na auditoria | **3 ciclos de correção sem fechar 100% conforme** → pausa e notifica | Mesmo princípio do teto por ticket, em escala de bloco |

## Decisões — Escopo do v1

| # | Pergunta | Decisão | Racional |
|---|---|---|---|
| 3/10 | Projeto piloto para validar a Factory | Já definido pelo usuário — **não revelado nesta sessão** | — |
| 4 | Linguagem suportada no v1 | **Só Python** | Reduz superfície de erro em paralelo à validação do pipeline; permite revisão real do código gerado |
| 5 | Uso pessoal vs. distribuição | **Uso exclusivamente pessoal** | Simplifica a tela de setup/pré-requisitos; empacotamento pra terceiros fica pra se/quando virar produto |
| 9 | Deploy do projeto gerado | **Fora do escopo do v1** | Deploy tem variáveis próprias por tipo de projeto que merecem pipeline à parte |
| 16 | Criação automática de repositório remoto no GitHub | **Não — só `git init` local** | Evita conceder um escopo de permissão (criação de repo) sem necessidade real ainda |
| 17 | Repertório de skills/MCP no dia 1 | **Genérico** (git, filesystem, browser-use) | Ferramentas específicas de stack dependem do piloto, ainda não revelado |

## Decisões — Infraestrutura & Segurança

| # | Pergunta | Decisão | Racional |
|---|---|---|---|
| 8/14 | Contas no round-robin do 9Router | **4 contas**, nos provedores **nvidia, groq e google** | Define o throughput real do "custo zero"; 3 provedores distintos dão fallback de verdade, não só entre contas do mesmo provedor |
| 12 | Políticas mínimas do Agente Auditor (agent-governance-toolkit) | **(a)** sandbox sem acesso de rede fora do 9Router · **(b)** allowlist de licença (MIT/BSD/Apache) pra dependência nova · **(c)** limite de tokens por ticket | Baseline barato de configurar (via OPA/Rego) que cobre os riscos mais óbvios de um agente autônomo sem supervisão linha a linha |
| 13 | Autenticação no Painel Web Local | **Nenhuma — bind só em `127.0.0.1`** | Uso pessoal, rodando local; senha seria complexidade sem ganho real |
| 19 | Escopo de "chave de API exclusiva por projeto" | Vale só pro **token do GitHub** (PAT *fine-grained*, por repositório). As chaves do **9Router ficam em pool compartilhado** entre todos os projetos | Isolar as chaves do 9Router por projeto reduziria as opções de fallback de cada um — o oposto do que o round-robin existe pra entregar; o risco de vazamento ali é menor, já que essas chaves nunca tocam o código do projeto |

## Decisões — Interface & Notificação

| # | Pergunta | Decisão | Racional |
|---|---|---|---|
| 6 | Validação humana nos checkpoints | **Botão como ação principal + chat disponível** | Rápido no caminho feliz, mas sem perder a opção de pedir ajuste em linguagem natural |
| 7 | Framework do Painel Web Local | **FastAPI** | WebSocket nativo cobre status em tempo real; mesma stack que o usuário já programa |
| 15 | Canal de notificação em pausa/escalonamento | **Só notificação nativa do SO, sem canal remoto** | Consistente com o padrão de manter o v1 leve; canal remoto (ex. Telegram) vira item de fase futura se sentir falta |

## Itens explicitamente adiados (não é lacuna — é escolha)

- Projeto piloto (existe, não revelado)
- Temporal (trade por SQLite reversível depois)
- Canal de notificação remoto
- Criação automática de repositório no GitHub
- Deploy automático do projeto gerado
- Ferramentas de skill específicas de stack (dependem do piloto)
