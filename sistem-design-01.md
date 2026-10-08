# Factory — System Design

## 1. Requisitos

### Funcionais
- Interrogar o usuário (Fase 1), produzir contrato (`CONTEXT.md` por bloco) e fatiar em tickets.
- Executar tickets autonomamente (Fase 2): TDD, sandbox, commit por ticket.
- Auditar blocos periodicamente contra o contrato (Fase 3), com política de governança.
- Reportar tempo restante e custo evitado (Fase 4).
- Fechar o projeto só com aprovação humana explícita (Fase 5).
- Gerar/atualizar `README.md`, `CHANGELOG.md` e tag semver como prática padrão, não opcional.

### Não-funcionais
- Custo zero: só modelos/ferramentas gratuitas ou self-hosted.
- Sobreviver a execuções de até 12h+ sem perder estado.
- Uso local, single-user — latência e throughput limitados pelo que o free tier aguenta, não por SLA.

### Restrições
- Solo dev, stack Python conhecida.
- Só Python como linguagem de saída no v1.
- 4 contas no 9Router, em 3 provedores (nvidia, groq, google).
- Sem autenticação no painel (bind `127.0.0.1`).
- Um projeto ativo por vez.

## 2. Design de Alto Nível
┌───────────────────────────────┐
│ Painel Web Local (FastAPI) │ ← bind 127.0.0.1, sem auth
│ REST + WebSocket │
└──────────────┬─────────────────┘
│ dispara / escuta status
┌──────────────▼─────────────────┐
│ Orquestrador (Python) │
│ lê/escreve estado no SQLite │
└───┬─────────┬──────────┬───────┘
│ │ │
┌───▼───┐ ┌───▼─────┐ ┌──▼──────────────────┐
│CrewAI │ │OpenHands │ │agent-governance- │
│Fase 1 │ │(Docker) │ │toolkit — Fase 3 │
│Arquit.│ │Fase 2 │ │Policy Engine + │
│Planej.│ │ticket │ │Auditor │
└───┬───┘ └────┬─────┘ └──┬──────────────────┘
│ │ │
└────┬─────┴────┬─────┘
│ │
┌─────▼───────────▼─────┐
│ 9Router │
│ 4 contas / 3 provedores│
│ nvidia · groq · google │
│ fallback + compressão │
└───────────┬─────────────┘
│
┌─────────▼──────────┐
│ Modelos gratuitos │
│ (lista rotativa) │
└─────────────────────┘

Repositório do projeto gerado:
/CONTEXT.md (um por bloco)
/src, /tests
/README.md ← auto-atualizado
/CHANGELOG.md ← conventional commits
tags semver (commitizen / python-semantic-release)

**Fluxo de dados**: usuário → interrogatório (Fase 1) → `CONTEXT.md` + fila de tickets no SQLite → loop por ticket no OpenHands/Docker, gateway via 9Router → a cada bloco concluído, auditoria (governance toolkit) → se conforme: tag semver + README/CHANGELOG atualizados → telemetria do 9Router alimenta estimativa de tempo/custo → Fase 5 fecha com aprovação humana.

**Contratos internos**: Painel Web Local fala com o Orquestrador só por REST (ações) + WebSocket (status ao vivo) — nenhum componente de frontend chama CrewAI/OpenHands/9Router diretamente.

**Armazenamento**: SQLite (estado de projetos/blocos/tickets), filesystem (`.md` de contexto, logs JSON lines), `.env` (secrets, nunca committado).

## 3. Deep Dive

### Modelo de dados (SQLite)
| Tabela | Campos principais |
|---|---|
| `projects` | id, name, repo_path, status, current_phase, created_at |
| `blocks` | id, project_id, name (backend/frontend/db/segurança...), status, audit_cycle_count |
| `tickets` | id, block_id, description, status, attempt_count, started_at, time_spent_seconds |
| `audit_reports` | id, block_id, cycle_number, result (conforme/não conforme), details_path, created_at |

### Endpoints internos (Painel ↔ Orquestrador)
- `GET /projects`, `GET /projects/{id}`
- `POST /projects` (criar)
- `POST /projects/{id}/approve` / `.../reject` (checkpoints humanos)
- `WS /projects/{id}/stream` (status ao vivo de ticket/agente)

### Máquina de estados — erro e retry
Ticket: pending → running → done
│
├─ fail → retrying (attempt++)
│ │
│ └─ attempt > 5 OU tempo > 30min → escalated (pausa + notifica)
Bloco: in_progress → audited
│
├─ conforme → approved (tag semver + README/CHANGELOG)
│
└─ não conforme → correcting (cycle++)
│
└─ cycle > 3 → escalated (pausa + notifica)

(Teto por ticket: 5 tentativas ou 30min. Teto por bloco: 3 ciclos. Ambos decididos na sessão de grilling.)

### Fila/concorrência
Fila simples em SQLite, FIFO, um projeto ativo por vez — suficiente dado que o gargalo real é o free tier, não CPU local.

## 4. Escala & Confiabilidade

- **Estimativa de carga**: free tier típico gira em torno de ~20 req/min por conta, com teto diário entre ~50 e ~200 requisições (varia por provedor/momento). Com 4 contas em 3 provedores, o teto diário agregado é o fator limitante, não o por-minuto — na prática, algo entre dezenas de tickets/dia dependendo de quantas chamadas cada ticket exige. Número otimista, trate como ordem de grandeza, não garantia.
- **Horizontal vs. vertical**: não se aplica — single machine, single user, por design. Não é meta do v1 escalar além disso.
- **Failover**: na camada de modelo, o 9Router já cascateia entre contas/provedores. Na camada de workflow, o SQLite dá retomada após crash (mais frágil que um motor dedicado como Temporal — trade-off aceito conscientemente no v1).
- **Ponto único de falha**: o próprio 9Router. Se cair, a Factory para. Mitigação do v1: nenhuma automática — documentar um caminho manual de fallback (chamar um provedor direto) pra destravar na mão se precisar.
- **Monitoramento**: notificação nativa do SO nos pontos de pausa/escalonamento + dashboard do 9Router para tokens/custo. Sem alerting externo (coerente com uso pessoal/local).

## 5. Trade-offs

| Decisão | Ganho | Custo aceito | Quando revisitar |
|---|---|---|---|
| SQLite em vez de Temporal | Menos infra pra rodar/aprender agora | Retomada após crash mais frágil, sem replay | Se os crashes em loops longos virarem dor real |
| OpenHands em vez de orquestrador caseiro | Ciclo completo já pronto, menos código próprio | Secrets reportados como imaturos + CVE corrigido em 03/2026 — mitigado mantendo tudo no Docker | Se precisar de controle mais fino sobre o loop interno |
| Sem autenticação no painel | Zero complexidade extra | Só seguro enquanto for 100% local | No dia em que quiser acessar de outro aparelho na rede |
| Repertório de skills genérico | Não trava decisão sem saber o piloto | Fase 1 pode pedir uma skill que ainda não existe | Assim que o piloto for revelado |
| Um projeto ativo por vez | Simplicidade, bate com o gargalo real (free tier) | Sem paralelismo entre projetos | Se o teto diário dos provedores deixar de ser o limitante |
