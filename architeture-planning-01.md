# Factory — Planning

## Objetivo
Automatizar o processo de desenvolvimento de projetos por meio de agentes operando em loop, de forma praticamente "ilimitada", usando somente ferramentas gratuitas (ou self-hosted sem custo).

## Ferramentas

| Camada | Ferramenta | Papel |
|---|---|---|
| Gateway de LLM | **9Router** (self-hosted) | Proxy compatível com OpenAI; fallback automático pago → orçamento → gratuito entre 40+ provedores; compressão de contexto (RTK Token Saver); dashboard de uso/tokens já embutido — alimenta a telemetria da Fase 4 |
| Orquestração multiagente | **CrewAI** (protótipo) → **LangGraph** (produção) | CrewAI mapeia direto os papéis (Arquiteto/Planejador/Programador/Revisor) pra validar rápido; LangGraph entra quando o loop precisa persistir estado entre pausas longas de validação humana |
| Agente Programador | **OpenCode** (principal) + **Aider** (fallback) | OpenCode: suporte a MCP, multiagente, arquitetura cliente/servidor pensada pra Docker/execução remota. Aider: integração git mais profunda, melhor pra refactors grandes e lineares |
| Sandbox | **Docker** | Container descartável por ticket, sem acesso à rede/host, comandos limitados |
| Skills/capacidades reutilizáveis | **Servidores MCP existentes** + pasta `skills/` com `SKILL.md` por playbook | Repertório pré-pronto (git, filesystem, browser, db, setup de stacks comuns) — evita reconstruir integrações que já existem |
| Versionamento | **Git + GitHub** | Commits automáticos via OpenCode/Aider, um commit por ticket |
| Vault de contexto | `.md` no repositório de cada projeto gerado, em convenção compatível com Obsidian (frontmatter + wikilinks) | Reaproveita um fluxo que você já usa |
| Interface da Factory | **Painel Web Local** (backend leve em Python, ex. FastAPI) que dispara comandos via CLI nos bastidores | Mesma stack que você já programa — nada novo pra aprender só pra isso |

## Método
1. Atalho abre o executável → checagem de pré-requisitos (python, docker, git, wifi, etc.).
2. Primeira tela do Painel Web Local: configuração de API Keys (pula se já configuradas).
3. Com keys + permissões + ferramentas ok, tela inicial é **"Projetos"**: entrar em projeto concluído/em desenvolvimento ou criar novo.
4. Projeto novo ou importado (não feito pela Factory) → indicar repositório + forma de conexão.
5. Se importado: agente faz leitura do estado atual do projeto (adapta/pula etapas conforme o que já existe).
6. Abre a página de desenvolvimento do projeto: chat, visualização dos agentes em execução, fase atual, estimativa de tempo, custo evitado (via 9Router), barra de progresso.
7. Toda ação do backend (orquestrador CrewAI/LangGraph, chamadas ao OpenCode, start/stop de containers Docker) roda via CLI, acionada e monitorada pelo Painel Web Local.

## Critérios
1) Uso do projeto compreendido por meio de contexto + interrogatório.
2) Requisitos do projeto compreendidos.
3) Alinhamento de resultados.
4) Produção de um protótipo para garantir o alinhamento.
5) Validação humana explícita de "contexto" para iniciar a próxima fase.
6) Apresentação de arquitetura + ferramentas indicadas (sempre custo zero) — ver tabela de Ferramentas acima.
7) Validação de arquitetura.
8) Criação dos contratos do projeto e fatiamento do escopo.
9) Separação das ferramentas e tarefas (repertório de skills por bloco).
10) Validação humana.
11) Start do Loop Engineering de criação.
12) A cada "grande bloco" completo ou 12h de execução, auditoria para confirmar integridade/coerência.
13) Se a auditoria encontrar erros, correção prioritária até controle total reestabelecido.
14) Estimativa de tempo restante + custo evitado por uso de tokens.
15) Software pronto → avaliação do usuário (pode solicitar mudanças e voltar etapas).
16) Até aprovação final → projeto "encerrado".

> Sempre usar `.md` nos repositórios dos projetos para contexto/specs. Documentação organizada e intuitiva por diretório. Preferir backend completo antes do frontend.

## Fases

### Fase 1 — Pré-Alinhamento e Especificação ("O Cérebro")
- **Interrogatório de Arquitetura**: agente com persona de Arquiteto Sênior, guiado por máquina de estados, entrevista o usuário sobre banco de dados, regras de negócio e segurança antes de codificar.
- **Contrato do Projeto (Spec)**: escopo aprovado vira `CONTEXT.md` por parte do projeto. Agentes de programação passam a seguir estritamente o contrato.
- **Fatiamento do Escopo**: Agente Planejador quebra o projeto em tickets independentes por bloco (segurança, frontend, backend, db...), mantendo as requisições à IA curtas.
- **Interpretação de Ferramentas**: um agente cruza cada bloco de tickets com o repertório de skills (MCP servers + `skills/SKILL.md`) e seleciona o que cada bloco pode usar.

### Fase 2 — Desenvolvimento Agêntico ("A Fábrica")
- **Motor de Inteligência na Nuvem**: requisições saem via 9Router, que cascateia entre modelos gratuitos de ponta disponíveis no momento (lista rotativa — hoje inclui DeepSeek, Qwen, Gemini Flash, entre outros) e faz fallback automático quando um provedor bate rate limit.
- Agentes já com contexto aprofundam Specs e Testes (harness) por ticket, priorizando meios de identificar quando um código "não é suficiente".
- **TDD**: OpenCode (ou Aider) lê o harness do ticket atual e itera o código real até o teste passar.
- **Sandbox**: todo script roda em container Docker descartável, sem acesso ao host, comandos limitados.
- **Revisão em Contexto Limpo**: logs de execução + código + requisitos originais vão pro Agente Revisor, que audita só contra o contrato definido na Fase 1.

### Fase 3 — Auditoria & Governança do Loop
- A cada bloco completo ou 12h de execução contínua, o **Agente Auditor** roda uma checagem objetiva por item: testes do bloco passando, specs do `CONTEXT.md` cumpridas, dependências entre blocos não quebradas.
- Gera um relatório binário (conforme/não conforme) por item.
- Se houver não-conformidade, o loop **pausa** e prioriza a correção antes de liberar novos tickets (critério 13).

### Fase 4 — Telemetria & Estimativas
- Consome os dados que o **dashboard do 9Router** já coleta (tokens, provedor usado, taxa de fallback) para estimar tempo restante e "custo evitado" (tokens consumidos × preço do modelo pago equivalente).
- Alimenta a barra de progresso e os indicadores da tela de desenvolvimento do projeto.

### Fase 5 — Validação Final & Encerramento
- Tela de revisão do software completo.
- Usuário pode aprovar, ou solicitar mudanças — nesse caso o projeto reabre num ponto específico do loop, não necessariamente do zero.
- Aprovação explícita marca o projeto como **encerrado**.
