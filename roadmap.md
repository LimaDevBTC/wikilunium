# Roadmap — desenvolvimento da lunium-api

> Plano de desenvolvimento do `lunium-api` (**cash-out** headless determinístico —
> ADR-016): em que ordem construir, do que cada fase depende e quando ela está
> "pronta". Não substitui os ADRs — **executa** o que eles decidiram.

> **STATUS: atualizado em 2026-06-25**

Ponto de entrada do projeto continua sendo o [`Lunium.md`](Lunium.md). Este plano
assume tudo que está fechado lá (§4 escopo; ADR-001…009). **Sem datas**: trata de
sequência e dependências; estimar com o time.

## Princípios que o plano respeita
- **Decisão primeiro, código depois.** Mudou algo? ADR aqui, depois código lá.
- **MVP é cash-out 100% determinístico** (ADR-016/003). Nada de camada agêntica.
- **Segurança/risco/auditoria são pré-condição de merge** (princípio 4 + ADR-008).

---

## Faixa B — Burocracia (começa JÁ, em paralelo — não é código, §6)
Bloqueia fases adiante; rodar desde o dia 1.
- [ ] **DECISÃO DA ENTIDADE (CNPJ) — pré-condição do KYB.** Irreversível (1 conta
  por CNPJ; não converte PF→PJ). Está parada na pauta de reunião — é o gargalo do
  **caminho crítico real** (a Fase 3 depende dela e a aprovação do KYB não está sob
  nosso controle). Resolver ANTES de tudo. Ver [`meetings/pauta-decisoes-pendentes.md`](meetings/pauta-decisoes-pendentes.md).
- [ ] **Postura de compliance/KYC-AML (ADR-012)** — quem faz o KYC, registro VASP,
  PLD/COAF. Bloqueia o go-live (Fase 3 com dinheiro real). Pauta + assessoria.
- [ ] Conta **SmartPay** (off-ramp do MVP): credenciais, webhook de payout, redes/tokens, limites, prazo de liquidação, **pré-funding de BRL?** → desbloqueia spec/adapter SmartPay.
- [ ] Conta **MEXC** institucional (KYB no CNPJ certo, **withdraw habilitado** p/ sacar USDT à SmartPay). → desbloqueia o caminho **convert**. Ver [`runbooks/onboarding-mexc-broker.md`](runbooks/onboarding-mexc-broker.md). **Nota ADR-017:** sub-contas não necessárias; conta master única suficiente.
- [ ] Domínio + repos + chaves novas. **Zero reaproveitamento do ambiente antigo.**
- [ ] (Fase 2) Conta **Eulen** — só quando o cash-in entrar.

---

## Faixa A — Construção

### Fase 0 — Fechar o brain (pré-código)
**Objetivo:** eliminar os stubs que bloqueiam a geração de código; sair com a fonte de verdade preenchida.
**Depende de:** parcialmente da Faixa B (specs de provedor precisam das contas).
**Entregáveis:**
- [x] ADR-009 — runtime/framework (TypeScript/Node + NestJS).
- [x] ADR-016 (pivot p/ cash-out) · ADR-015 (VPS + IP egress) · ADR-012 (compliance — Proposto) · ADR-013 (outbox). ADR-011/014 são cash-in/fase 2.
- [~] [`domain/state-machine.yaml`](domain/state-machine.yaml) — **cash-out** (FAST + convert), estados (incl. exceção: `MANUAL_REVIEW`, `REFUNDING_CRYPTO/REFUNDED`), transições, `failure_state` e **`idempotency_key` por etapa DEFINIDOS**; faltam só **valores** (timeouts/retries/confirmações — Faixa B).
- [~] [`providers/_capability-contract.yaml`](providers/_capability-contract.yaml) — `off_ramp` (SmartPay) + `exchange` (sell/depósito/saque), `failure_modes`, `cross_cutting` **definidos**; **números** de `limits`/`rate_limits` dependem das contas (Faixa B).
- [ ] [`providers/smartpay.md`](providers/smartpay.md) — endpoints, webhook de payout, redes/tokens, limites (da conta nova).
- [ ] [`providers/mexc.md`](providers/mexc.md) — endpoints reais, auth HMAC, limites, redes de depósito/saque (extrair da demo MEXC, §7).
- [~] [`domain/money-rules.md`](domain/money-rules.md) — **decidido:** tipo monetário (decimal/bigint, nunca float), precisão por ativo (catálogo MEXC), truncar a favor do operador, under-fill na venda; **ticket mínimo e as duas taxas (FAST/convert) dependem das integrações/negócio** (Faixa B + pauta).

**Pronto quando:** `state-machine.yaml` e `_capability-contract.yaml` sem TODO **crítico** (os TODO restantes são valores operacionais da Faixa B, não estrutura); specs de provedor com endpoints/limites confirmados.

### Fase 1 — Scaffold da lunium-api (esqueleto DDD) ✅
**Objetivo:** repositório de código vivo, vazio mas com a arquitetura certa.
**Depende de:** Fase 0 (contract + runtime).
**Entregáveis:**
- [x] Repo `lunium-api`: NestJS + TypeScript, **estrutura por domínio** (não por camada) — ADR-006. [github.com/LimaDevBTC/lunium-api](https://github.com/LimaDevBTC/lunium-api)
- [x] **Ports** declarados como interfaces TS (`ExchangePort`, `OffRampPort`, `PersistencePort`, `QueuePort`).
- [x] **PostgreSQL** + Prisma schema + migrations (`CashOut`, `CashOutEvent`, `OutboxJob`) — ADR-004.
- [x] **BullMQ/Redis** ligado (ADR-007); `OutboxRelay` + `CashOutWorker` idempotentes.
- [x] **Harness de testes** + fakes (`FakeMexc`, `FakeSmartPay`, `FakePersistence`, `FakeQueue`); 20 testes passando.

**Pronto quando:** build verde, CI rodando, job dummy idempotente passa em teste de integração. ✅

### Fase 2 — Adapters de provedor (atrás dos ports) ✅ (com fakes) / 🔄 integração real pendente
**Objetivo:** SmartPay e MEXC como adapters, testáveis sem bater na API real.
**Depende de:** Fase 1 + specs da Fase 0 + contas (Faixa B).
**Entregáveis:**
- [x] Adapter **SmartPay** (off_ramp): `createPayoutOrder` (encode-memo + endereço por rede), `retryPayout` (no-op), `getPayoutStatus`, `verifyWebhook` (apiKey). + **fake**. JWT cacheado 55 min. (commit `18b1f4b`)
- [x] Adapter **MEXC** (exchange — **ADR-017**: conta master + txId, sem sub-contas): `getCatalog`, `getDepositAddress`, `getDeposit`, `sellSpot` (IOC — ADR-001), `getOrder`, `withdrawOnchain`, `getWithdrawal`, `getBalance`. + **fake**.
- [x] Erros normalizados (`MexcApiError` + `MEXC_CODE`); idempotência via `clientOrderId`/`withdrawOrderId`.
- [x] Fakes cobrem cenários de falha do ADR-008: webhook duplicado, fill parcial, depósito divergente. *(81 testes passando)*
- [ ] Testes de integração contra real (MEXC + SmartPay) — aguarda credenciais (Faixa B). Setup SmartPay: `POST /v1/application/create-source-address` + webhook `PIX_OFFRAMP`.

**Pronto quando (fakes):** ✅ | **Pronto quando (real):** depende da Faixa B.

### Fase 3 — Orquestração do cash-out (máquina de estados sobre a fila) ✅ (com fakes)
**Objetivo:** o fluxo ponta a ponta, determinístico e recuperável.
**Depende de:** Fase 2 + MEXC com **withdraw habilitado** + conta SmartPay (Faixa B) para validar contra APIs reais.
**Entregáveis:**
- [x] Máquina de estados de domínio implementando [`state-machine.yaml`](domain/state-machine.yaml), **incl. caminhos de exceção** (`MANUAL_REVIEW`, `REFUNDING_CRYPTO/REFUNDED`). (`CashOutOrchestrator`)
- [x] Cada transição = **job BullMQ idempotente**, **1 transação ACID** (ADR-004/007), com **outbox** (ADR-013), retry/backoff e `failure_state`. (`PrismaPersistenceAdapter`, `OutboxRelay`)
- [x] **Webhook SmartPay** como fonte de eventos (FAST/payout). (`CashOutController`)
- [x] **Reaper** — detecta `AWAITING_DEPOSIT` expirado e jobs stuck (SELLING/FORWARDING > 30 min). (`ReaperWorker`)
- [x] Fluxo E2E **FAST** e **convert** verdes com fakes (20 testes).
- [ ] Validação com APIs reais (sandbox MEXC + SmartPay) — aguarda contas (Faixa B).

**Pronto quando (fakes):** ✅ | **Pronto quando (real):** depende da Faixa B.

### Fase 4 — Reconciliação + visibilidade do operador ✅ (parcial)
**Objetivo:** fechar o MVP (§4) e dar ao Guilherme visão da operação (ADR-010).
**Depende de:** Fase 3.
**Entregáveis:**
- [x] `ReconciliationService`: detecta cash-outs presos em estado intermediário além do TTL (por estado).
- [x] **Dashboard de reconciliação** headless — `GET /admin/reconciliation` (divergências + uso do dia + flag circuit-breaker). Protegido por `ADMIN_TOKEN`.
- [x] **Endpoints de operador/admin**: `GET /admin/cash-outs` (filtro/paginação) + `GET /admin/cash-outs/:id/events` (trilha ACID completa).
- [x] **Bot Telegram thin** (alertas unidirecionais) — `AlertService` fire-and-forget; hooks no `ReaperWorker` e `CashOutOrchestrator`; no-op sem credenciais.
- [x] Reconciliação formal: `ReconciliationService` implementado + [`runbooks/reconciliation.md`](runbooks/reconciliation.md) preenchido com procedimento completo e TTLs reais.
- [x] `POST /admin/cash-outs/:id/action` — ações de resolução de `MANUAL_REVIEW`: `retry_sell`, `retry_forward`, `retry_payout`, `refund`, `complete_refund`, `write_off`. Campo `reason` obrigatório para auditoria.
- [ ] Bot Telegram com **consultas** interativas (read-mostly) — alertas prontos; queries ficam para após o go-live.

**Pronto quando:** divergências detectáveis e auditáveis; Guilherme recebe alertas e consulta a operação.

### Fase 5 — Hardening, segurança e continuidade (pré-produção) 🔄 Em andamento
**Objetivo:** merge-ready e operável sem o autor original.
**Depende de:** Fase 3 (em paralelo com a 4).
**Entregáveis:**
- [x] `Dockerfile` multi-stage (node:20-alpine) + `entrypoint.sh` (executa `prisma migrate deploy` antes de subir).
- [x] `ApiTokenGuard` (`CLIENT_API_TOKEN`) nos 3 endpoints públicos; webhook SmartPay e `/admin/*` isentos.
- [x] Rate limiting (`@nestjs/throttler`): 5/min em `POST /cash-outs`, 10/min em `/accept`, 60/min em `GET /:id`; `THROTTLE_LIMIT` env para override.
- [x] [`security/threat-model.md`](security/threat-model.md) + [`security/controls.md`](security/controls.md) revisados e alinhados ao ADR-017.
- [x] Runbooks [`key-rotation.md`](runbooks/key-rotation.md) e [`incident-stuck-order.md`](runbooks/incident-stuck-order.md) preenchidos com procedimentos reais.
- [x] Confirmação explícita de movimento de dinheiro: `reason` obrigatório em `POST /admin/cash-outs/:id/action`; logs via `Logger.warn` para auditoria.
- [ ] **Security review** aprovado (pré-condição de merge — princípio 4).

**Pronto quando:** security review passa; runbooks operacionais completos; checklist de go-live fechado.

### Gate de go-live — canário com dinheiro real (limites duros)
**Objetivo:** primeira transação real sem expor capital/clientes a um bug de fluxo.
**Depende de:** Fase 5 + compliance definido (ADR-012) + KYB com withdraw (Faixa B).
**Entregáveis:**
- [ ] **Allowlist** de usuários (só o time no começo) e **ticket mínimo/baixo**.
- [x] **Limite diário duro** de volume e nº de operações (`DAILY_MAX_OPS`, `DAILY_MAX_BRL`) — implementado em `DailyLimitService`.
- [x] **Circuit-breaker manual** (`CASHOUT_ENABLED=false`) — fecha todas as novas cotações com 503.
- [ ] Período de **shadow/canário** observado (reconciliação batendo 100%: depósito ↔ venda/saque ↔ payout) antes de abrir ao público.

**Pronto quando:** N operações reais pequenas concluídas e reconciliadas sem divergência; limites prontos para subir gradualmente.

---

## Integração com o front (`vendacripto-app`, Cliente #001 — ADR-005)
Paralelo a partir da Fase 2/3: estabilizar o **contrato da API headless** que o front consome; **cutover de DNS** para `vendacripto.io` no fim. Front é repo separado; a infra fica client-agnostic.

## Explicitamente FORA do MVP (não fazer agora)
**Cash-in (comprar cripto — Eulen/PIX-in)**, camada agêntica (fase 2 — ADR-003), múltiplos provedores simultâneos, troca automática de corretora, PIX próprio, moeda própria. A arquitetura já nasce plugável para isso (capabilities/ports), mas o código v1 é fatia fina.

**Planos futuros registrados:**
- **Canal de venda no Telegram** — bot transacional próprio e, como **produto-brinde B2B**, entregue pronto a parceiros/clientes Lunium rodarem sem ter app próprio. É só mais um cliente da api headless (ADR-005/010).
- **Operador agêntico** — controle da operação por linguagem natural (evolução do bot operador determinístico da Fase 4), com os guard-rails do ADR-003.

## Como manter este plano
Mudou escopo ou sequência? Atualize aqui **e** registre um ADR se for arquitetural. Marque as caixas conforme entrega; ao fechar uma fase, anote no [`CHANGELOG.md`](CHANGELOG.md).
