# Controles de segurança

> Controles concretos (acesso, segregação, auditoria, continuidade) que
> implementam o modelo de ameaças.

> **STATUS: revisado em 2026-06-23**

## Controle de acesso e segredos
- Segredos em cofre, **fora do código**; `.gitignore` bloqueia `.env`; só `.env.example` é commitado.
- Rotação de chaves — [`../runbooks/key-rotation.md`](../runbooks/key-rotation.md); cadência recomendada: 90 dias em routine, imediato em suspeita.
- **Chaves de privilégio mínimo:** chaves MEXC separadas por função (ler / negociar / **sacar**); chave com permissão de saque é a "joia da coroa" — só no worker do VPS.
- **Allowlist de endereço de saque** na MEXC: mesmo com a chave comprometida, o saque só vai para o endereço SmartPay pré-aprovado.

## Autenticação por camada
- **Clientes da API** (`vendacripto-app`): `CLIENT_API_TOKEN` (Bearer) verificado pelo `ApiTokenGuard` em `POST /cash-outs`, `POST /cash-outs/:id/accept` e `GET /cash-outs/:id`. Sem token configurado = aberto (dev). Rotacionar em [`key-rotation.md`](../runbooks/key-rotation.md).
- **Endpoints de admin** (`/admin/*`): `ADMIN_TOKEN` (Bearer) verificado no `AdminController`. Protege listagem, eventos e reconciliação. Isentos de rate limiting.
- **Webhook SmartPay** (`POST /webhooks/smartpay`): autenticado por HMAC (`x-signature`) via `offRamp.verifyWebhook`. Isento de token de cliente e rate limiting.
- **MEXC**: HMAC-SHA256 via `MexcHttpClient` (`X-MEXC-APIKEY` + `signature` no query string). Nenhum segredo MEXC chega ao front.

## Rate limiting (proteção de abuso)
- `@nestjs/throttler` global: `POST /cash-outs` → 5 req/min/IP; `POST /accept` → 10/min/IP; `GET /:id` → 60/min/IP.
- Webhook SmartPay e `/admin/*` com `@SkipThrottle()` — fontes autenticadas por outros meios.
- Override via `THROTTLE_LIMIT` (env) para staging.

## Egress e IP allowlist de provedor (ADR-015)
- O keyholder (`lunium-api`) roda em **host persistente com IP estático**; esse
  **único IP** entra na allowlist de cada provedor (a Vercel/serverless não dá IP
  fixo barato — lição do btcnopix). O **front nunca chama o provedor** e **nenhum
  cliente precisa de IP fixo**.
- IP allowlist é **uma camada** entre várias (chaves de privilégio mínimo +
  allowlist de saque + HMAC + cofre), não o pilar único.

## Segregação e rastreabilidade de fundos
- **Conta master única** (ADR-017): todos os depósitos CONVERT chegam ao mesmo endereço MEXC; cada depósito é identificado pelo `txId` on-chain — sem ambiguidade de a quem pertence.
- `ReconciliationService` detecta automaticamente cash-outs presos em estados intermediários além do TTL (AWAITING_DEPOSIT por `expiresAt`; demais por TTL por estado). Operador vê via `GET /admin/reconciliation`.
- Trilha ACID completa de cada transição de estado em `CashOutEvent` — auditável via `GET /admin/cash-outs/:id/events`.

## Idempotência e atomicidade
- Chave de idempotência por etapa (ADR-007); `hasApplied()` faz early-exit se o evento já foi aplicado.
- Cada transição + event + outbox gravados numa **única transação ACID** (ADR-013): sem ordem travada nem job fantasma.
- `newClientOrderId="{id}:sell"` na MEXC: venda duplicada retorna o mesmo resultado sem nova execução.
- `withdrawOrderId="{id}:forward"` na MEXC: saque duplicado é no-op seguro.

## Compliance / screening
- **Gate de screening** reservado **antes do payout** em BRL (`screening_passed` —
  seam). Crítico no cash-out (cripto→fiat é vetor de lavagem). Política (KYC/AML/
  sanções, quem executa) **pendente** — ADR-012 / pauta. Quando fechar, vira
  controle ativo (não no-op).

## Auditoria, logs e alertas
- Trilha de auditoria por transição de estado em `CashOutEvent` (tabela Postgres): `from`, `to`, `event`, `idempotencyKey`, `createdAt`. Retenção: definir política antes do go-live (recomendação: 2 anos para compliance PLD).
- `AlertService` (Telegram, fire-and-forget): 🚨 `FAILED`/`MANUAL_REVIEW`, ⚠️ `REFUNDING_CRYPTO`/stuck detectado pelo Reaper, ℹ️ quote expirada. No-op sem `TELEGRAM_BOT_TOKEN`.
- `ReaperWorker` roda a cada 60s: expira `AWAITING_DEPOSIT` além de `expiresAt`, move estados stuck além do TTL para `MANUAL_REVIEW`.

## Confirmação explícita de movimento de dinheiro
- Execução determinística; agente nunca move fundos (ADR-003). Ações do operador
  que movem dinheiro (refund/retry/write-off em `MANUAL_REVIEW`) exigem auth forte
  + confirmação + auditoria (ADR-010).

## Continuidade
- Doutrina, decisões e runbooks vivem neste brain (princípio 5). Onboarding de um
  novo responsável é via `Lunium.md` + runbooks — sem dependência de uma pessoa.
