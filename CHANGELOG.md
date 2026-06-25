# Changelog

Todas as mudanças relevantes deste brain são registradas aqui.
Formato baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/).

## [Não lançado]

### Allowlist canário + testes MexcAdapter (2026-06-25) — commit `8da1950`

**Allowlist canário — Gate de go-live**
- `CANARY_PIX_KEYS` env var (lista separada por vírgula): só as chaves PIX listadas conseguem criar cotações; qualquer outra recebe HTTP 403.
- Vazio = aberto para todos (comportamento normal pós-canário).
- Check executado após o circuit-breaker (`CASHOUT_ENABLED`) e antes do `DailyLimitService` — ordem de prioridade correta.
- 10 testes: modo aberto, chave permitida, bloqueio, trim de espaços, CPF numérico, ordem dos checks.

**Testes unitários MexcAdapter — 35 testes (fetch mockado)**
- `getCatalog`: filtro de redes desabilitadas, limites conservadores entre múltiplas redes, normalização MEXC→nosso padrão.
- `getDepositAddress`: mapeamento polygon→MATIC/solana→SOL/tron→TRX; tag/memo retornado quando presente.
- `getDeposit`: lookup por txId no histórico; status pending/confirmed; lança quando não encontrado.
- `sellSpot`: filled/canceled; idempotência — `DUPLICATE_CLIENT_ORDER` busca a ordem existente; normaliza `INSUFFICIENT_BALANCE` e `SYMBOL_HALTED`.
- `getOrder`: filledQty=USDT recebido, avgPrice calculado; edge case executedQty=0.
- `withdrawOnchain`: mapeamento de rede, addressTag; normaliza `ADDRESS_NOT_ALLOWLISTED` e `NETWORK_SUSPENDED`.
- `getWithdrawal`: mapeamento status 0-6 → domínio; txId; lança quando não encontrado.
- `getBalance`: asset encontrado/ausente; case-insensitive.
- `MexcHttpClient`: envelope `{code, msg}`, HTTP non-200, resposta não-JSON.
- 126 testes passando no total.

---

### SmartPayAdapter real — Frente 5 (2026-06-25) — commit `18b1f4b`

**SmartPayAdapter — implementação completa contra API WantsPay/Truther.sv**
- Autenticação JWT (`POST /v1/auth/login`) com cache de 55 min e renovação automática 5 min antes de expirar.
- `createPayoutOrder`: encode do memo via `POST /v1/application/encode-memo` (formato MemoDataEVM `"<pixKey>|refund:<refundAddress>"`); retorna endereço de depósito SmartPay por rede (Polygon/Solana/Tron) configurado via env.
- `getPayoutStatus`: `GET /v1/br/payments/off-ramp/status?txid=<txid>` — mapeia `SUCCESS → paid`, `FAILED|REFUNDED → failed`, demais → `paying`.
- `verifyWebhook`: verifica `apiKey` no header `x-signature`; extrai `payoutOrderId` de `reference` (cashOutId echoed) ou fallback para `txid`.
- `retryPayout`: no-op proposital — SmartPay não tem endpoint de retry explícito; o payout é re-tentado via fluxo normal ao receber USDT.
- 20 novos testes unitários com `fetch` mockado. 81 testes passando.

**Orchestrator — fix caminho CONVERT no FORWARDING**
- Para CONVERT, `exchange.withdrawOnchain` agora chama `offRamp.createPayoutOrder` primeiro para obter o endereço SmartPay correto (antes usava `depositAddress = MEXC address`, o que estava semanticamente errado).
- `payoutOrderId` e `depositAddress` (SmartPay) são persistidos atomicamente junto com `withdrawalId`.

**`providers/smartpay.md`** — documento completo: autenticação, fluxo off-ramp, encode-memo, status, webhook, env vars, erros normalizados, perguntas em aberto.

---

### Endpoints de ação admin (MANUAL_REVIEW) — Frente 4-B (2026-06-23) — commit `f118100`

**`POST /admin/cash-outs/:id/action`**
- 6 ações: `retry_sell`, `retry_forward`, `retry_payout`, `refund`, `write_off`, `complete_refund`.
- Campo `reason` obrigatório (auditoria em log via `Logger.warn`).
- Retorna `{ cashOutId, action, state }`. HTTP 422 para transição inválida; 404 para cash-out inexistente.
- Mapeamento `ACTION_TO_EVENT` canônico: `retry_sell → REVIEW_RETRY_SELL`, `refund → REFUND_DECIDED`, etc.
- 9 novos testes unitários. `@SkipThrottle()` no AdminController inteiro.

---

### Hardening pré-go-live — Frente 4-A (2026-06-22) — commit `97bcef5`

**Dockerfile + build de produção**
- `Dockerfile` multi-stage (node:20-alpine): stage `build` compila TypeScript; stage `runtime` instala só deps de produção.
- `entrypoint.sh`: executa `npx prisma migrate deploy` antes de `exec node dist/main.js` — prod nunca sobe sem migrations aplicadas.
- `.dockerignore`: exclui `node_modules/`, `dist/`, `.env`, `.env.*`, `test/`, `.git/`.
- `docker compose --profile full up api` sobe a API com as dependências saudáveis.

**ApiTokenGuard — auth dos endpoints públicos**
- `src/api/guards/api-token.guard.ts`: verifica `Authorization: Bearer <CLIENT_API_TOKEN>`.
- No-op quando `CLIENT_API_TOKEN` não está configurado (dev aberto por padrão).
- Aplicado em `POST /cash-outs`, `POST /cash-outs/:id/accept`, `GET /cash-outs/:id`.
- Webhook SmartPay (`POST /webhooks/smartpay`) e `/admin/*` ficam isentos (autenticados por HMAC e `ADMIN_TOKEN` respectivamente).

**Rate limiting via `@nestjs/throttler`**
- `ThrottlerModule` global: 5 req/min em `POST /cash-outs`, 10 em `/accept`, 60 em `GET /:id`.
- `@SkipThrottle()` em webhook SmartPay e todos os endpoints `/admin/*`.
- `THROTTLE_LIMIT` env para ajustar o default global (30 req/min) em staging.

**AlertService — bot Telegram thin (alertas unidirecionais)**
- `src/infrastructure/alert.service.ts`: `send(level, message)` fire-and-forget via `fetch` nativo (Node 20).
- Emojis por nível: 🚨 `error`, ⚠️ `warn`, ℹ️ `info`. Formato Markdown.
- Falha silenciosa com log — nunca bloqueia o fluxo principal.
- No-op quando `TELEGRAM_BOT_TOKEN` ou `TELEGRAM_CHAT_ID` não configurados.
- Hooks: `ReaperWorker` (expired → info, stuck → warn) + `CashOutOrchestrator` (FAILED/MANUAL_REVIEW → error, REFUNDING_CRYPTO → warn).
- 52 testes passando (+ 12 novos: `ApiTokenGuard` × 5, `AlertService` × 7).

---

### Operabilidade e limites — Frente 3 (2026-06-22) — commit `c2deee2`

**Circuit-breaker e limites diários**
- `CASHOUT_ENABLED=false` fecha todas as novas cotações com HTTP 503.
- `DailyLimitService`: verifica `DAILY_MAX_OPS` (nº de ops) e `DAILY_MAX_BRL` (volume BRL) antes de persistir cada cash-out. Zero = sem limite.
- Verificação ocorre no `POST /cash-outs` ANTES de salvar no banco — sem rollback necessário.

**Endpoints de operador/admin**
- `AdminController` protegido por `ADMIN_TOKEN` (Bearer): sem token configurado = aberto em dev.
- `GET /admin/cash-outs?state=&limit=&offset=` — lista com filtro e paginação (max 200).
- `GET /admin/cash-outs/:id/events` — trilha ACID de todos os eventos de um cash-out.
- `GET /admin/reconciliation` — snapshot: flag `cashoutEnabled`, `todayUsage` (ops + BRL), lista de `divergences`.

**ReconciliationService**
- Detecta cash-outs em estados intermediários parados além do TTL por estado: AWAITING_DEPOSIT por `expiresAt`; DEPOSIT_RECEIVED/SELLING/SOLD/PAYING_OUT = 30 min; FORWARDING = 60 min.
- Resultado ordenado por `stuckSinceMs` decrescente (mais velho primeiro).
- 40 testes passando (+ 20: `DailyLimitService` × 13, `ReconciliationService` × 7).

---

### Implementação da lunium-api — Frente 1 + 2A (2026-06-22)

**ADR-017 — Conta master + rastreamento por txId (supercede ADR-002)**
- Descoberto: limite de 30 sub-contas e API key por sub-conta inviabilizam ADR-002.
- Decisão: conta master única; depósitos identificados por txId on-chain.
- `subaccountId` removido do domínio, persistência e ports.
- Idempotência via `newClientOrderId` (venda) e `withdrawOrderId` (saque).
- Caminho de migração para Broker Program documentado (port não muda, só o adapter).

**Frente 1 — Camada de orquestração com fakes**
- `PrismaPersistenceAdapter`: `create` / `findById` / `applyTransition` — transação
  ACID única (CashOut update + CashOutEvent + OutboxJob) com idempotência P2002.
- `OutboxRelay`: polling 500ms, publica `OutboxJob` no BullMQ por `idempotencyKey`.
- `BullMqQueueAdapter`: `ConnectionOptions` via `REDIS_URL`.
- `CashOutOrchestrator`: guards determinísticos, dispatch de effects por nome, `hasApplied` early-exit.
- `QuoteService`: path FAST/CONVERT (USDT em polygon/solana/tron = FAST), validação
  catálogo, `truncateDown`, TTL 15 min, `expiresAt`.
- `CashOutController`: 4 endpoints (quote, accept, status, webhook SmartPay).
- `CashOutWorker` (BullMQ, roteamento por `job.name`) + `ReaperWorker` (expiração e stuck jobs).
- 20 testes E2E com `FakePersistence` + `FakeQueue` (FAST, CONVERT, idempotência, falhas).

**Frente 2A — `MexcAdapter` real**
- `MexcHttpClient`: HMAC-SHA256, header `X-MEXC-APIKEY`, parse de erros `{code, msg}`.
- `mexc.types.ts`: tipos brutos de todas as respostas MEXC v3.
- `MexcAdapter`: 8 métodos do `ExchangePort` (catálogo, depósito, venda IOC, saque, saldo).
- Normalização de redes: TRX↔tron, SOL↔solana, MATIC↔polygon, BTC↔btc.
- Idempotência no `sellSpot`: `-2010 DUPLICATE_CLIENT_ORDER` → busca por `origClientOrderId`.
- Erros mapeados: balanço insuficiente, par suspenso, rede suspensa, allowlist.

**Wiki (este repositório)**
- ADR-017 criado; ADR-002 marcado como superseded.
- `providers/mexc.md`: preenchido com endpoints reais, auth HMAC, normalização de
  redes, status de saque, tabela de erros.
- `Lunium.md`: repo `lunium-api` linkado, estado atual atualizado, ADR-002→017.
- `roadmap.md`: Fase 1 ✅, Fase 2 (MEXC ✅ / SmartPay ⏳), Fase 3 ✅ (com fakes).

### Pivot do MVP para cash-out (2026-06-21)

**Mudança de escopo (ADR-016)**
- O MVP passa de **cash-in** (comprar cripto) para **cash-out** (vender cripto →
  PIX via SmartPay): caminho **FAST** (USDT em Polygon/Solana/Tron direto p/ a
  SmartPay, ~10s) e **convert** (demais via MEXC: venda spot → USDT → SmartPay).
  É capital-light (cliente traz o ativo; SmartPay frente o BRL). **Cash-in vira fase 2.**
- `domain/state-machine.yaml` **reescrita** para o cash-out (QUOTE → AWAITING_DEPOSIT
  → [FAST: payout] | [convert: DEPOSIT → SELLING → SOLD → FORWARDING] → PAYING_OUT →
  COMPLETED; exceções `MANUAL_REVIEW`, `REFUNDING_CRYPTO/REFUNDED`).
- `providers/_capability-contract.yaml`: port `off_ramp` (SmartPay: create_payout_order,
  retry_payout, verify_webhook) + `exchange` (get_deposit_address, get_deposit,
  **sell_spot**, withdraw p/ SmartPay); `on_ramp` (Eulen) → fase 2.

**Providers**
- SmartPay → **provedor confirmado do MVP**; Eulen (PIX-in) → **fase 2**; MEXC muda
  de papel (recebe depósito + **vende** spot + saca USDT p/ a SmartPay).
- ADR-011 (estoque dois-mundos) e ADR-014 (reserva anti-oversell) → **cash-in/fase 2**
  (cash-out é capital-light, sem oversell).

**Alinhamento**
- Atualizados Lunium.md (§0/§3/§4/§5/§2/§6), roadmap (faixas/fases), money-rules
  (venda, taxas FAST/convert, truncar a favor do operador), glossary, security
  (threat-model/controls — webhook SmartPay, screening antes do payout, depósito
  divergente) e runbooks (reconciliação e incidente no fluxo de venda).

### Topologia de implantação + off-ramp FAST (2026-06-21)

**Adicionado**
- ADR-015 (topologia de implantação + IP de egress): keyholder (`lunium-api`) em
  host persistente com **um IP estático** na allowlist dos provedores (~US$5/mês,
  não US$100 da Vercel — lição do btcnopix/GetMoons); front sem chave nem IP;
  nenhum cliente precisa de IP fixo. Segurança por camadas (privilégio mínimo +
  allowlist de saque + HMAC + cofre), não só IP.
- `providers/smartpay.md`: doutrina do off-ramp com **modo FAST** (USDT em
  Polygon/Solana/Tron direto p/ a SmartPay, PIX em ~10s) e **modo convert** (demais
  moedas via MEXC). Escopo (MVP vs fase 2) a confirmar — pauta.
- `security/` (controls + threat-model): chaves de privilégio mínimo, allowlist de
  endereço de saque, IP de egress único; `glossary.md`: off-ramp/modo FAST,
  IP de egress, keyholder.

### Auditoria de engenharia (2026-06-12) — correções e novas decisões

**Adicionado**
- ADR-012 (compliance/KYC-AML — **Proposto**): reserva o gate de screening na
  máquina (`screening_passed`); política pendente de reunião, bloqueia o go-live.
- ADR-013 (outbox transacional): consistência fila↔banco; elimina ordem travada /
  job fantasma do dual-write (ADR-004 + ADR-007).
- ADR-014 (reserva de liquidez / available-to-promise): pré-cheque anti-oversell.
- `state-machine.yaml`: estados de exceção `WITHDRAW_BLOCKED`, `MANUAL_REVIEW`,
  `REFUNDING`, `REFUNDED`; caminho de under-fill (`FILL_UNDER_TOLERANCE`); chaves
  de idempotência definidas por etapa; seam de screening e re-cheque de rede.
- `money-rules.md`: regra de tipo monetário (decimal/bigint, nunca float JS) e
  tratamento explícito de under-fill.
- `_capability-contract.yaml`: método `on_ramp.refund_pix`; `cross_cutting`
  preenchido (idempotência, outbox, retry, normalização de erro).
- `roadmap.md`: reaper de estados travados na Fase 3; **gate de go-live canário**
  com limites duros; dependência crítica KYB↔decisão da entidade explicitada.

**Corrigido**
- Backfill de ADR-001, ADR-002 e ADR-003 (Contexto/Consequências/Alternativas
  estavam vazios — feria o princípio 5).
- Drift de nome `liquidity_failed` → `AWAITING_LIQUIDITY` (liquidity-model.md).
- Refund agora é representável na máquina (antes: `FAILED` terminal sem saída).
- `security/` (threat-model + controls): AML/screening, endereço sancionado,
  oversell, outbox; `glossary.md` e runbooks alinhados aos novos estados.

**Pendente de reunião** (movido para `meetings/pauta-decisoes-pendentes.md`):
política de compliance, parâmetros de reserva/oversell, tolerância de under-fill,
verificação MEXC (sub-contas/ToS) + risco de concentração, mecanismo de refund
(Eulen), TTL do quote, custo carregado das duas pontas p/ a taxa.
