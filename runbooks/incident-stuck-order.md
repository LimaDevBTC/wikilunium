# Runbook: Incidente — ordem travada

> O que fazer quando uma operação de cash-out trava num estado intermediário
> (ex.: venda não executou, saque on-chain travou, PIX não foi pago).
>
> **STATUS: atualizado em 2026-06-23**

---

## 1. Como você descobre

Dois mecanismos automáticos avisam antes do cliente reclamar:

**AlertService (Telegram):**
- ⚠️ `warn` — `ReaperWorker` detectou operação stuck além do TTL e moveu para `MANUAL_REVIEW`
- 🚨 `error` — `CashOutOrchestrator` transitou para `MANUAL_REVIEW` ou `FAILED` diretamente
- ⚠️ `warn` — `CashOutOrchestrator` transitou para `REFUNDING_CRYPTO`

**Reconciliação diária:**
- `GET /admin/reconciliation` retorna `divergences` com operações além do TTL por estado.

Se nenhum dos dois funcionou, pode ser: `TELEGRAM_BOT_TOKEN` não configurado (alerta vai só para o log do servidor) ou `ReaperWorker` parou de rodar.

---

## 2. Diagnóstico inicial

### 2.1 — Ver a trilha completa da operação

```
GET /admin/cash-outs/:id/events
Authorization: Bearer <ADMIN_TOKEN>
```

Ler a sequência de eventos de baixo para cima: o último evento revela onde parou. O campo `idempotencyKey` confirma que o evento foi processado (ou não).

### 2.2 — Identificar o estado atual

```
GET /admin/cash-outs/:id
Authorization: Bearer <ADMIN_TOKEN>
```

O campo `state` é a fonte de verdade. Estados relevantes para incidente:

| Estado | O que está acontecendo |
|---|---|
| `SELLING` parado > 30 min | Venda spot MEXC travada |
| `FORWARDING` parado > 60 min | Saque on-chain MEXC → SmartPay lento/travado |
| `PAYING_OUT` parado > 30 min | SmartPay não confirmou o PIX |
| `MANUAL_REVIEW` | Sistema detectou problema; exige decisão humana |

---

## 3. Por estado — diagnóstico e ação

### Estado: `SELLING`

**Causa provável:** par suspenso, fill parcial abaixo da tolerância (90%), ordem cancelada pela MEXC.

**Diagnóstico:**
1. Pegar `orderId` da trilha de eventos.
2. Checar diretamente na MEXC: status da ordem, `executedQty`, `status`.
3. Se `status = CANCELED` ou `executedQty < 90% do esperado` → under-fill ou cancelamento.

**Decisão:**
- Falha temporária da MEXC (par voltou a negociar) → `REVIEW_RETRY_SELL`
- Fill definitivamente abaixo → `REFUND_DECIDED` (devolve o que entrou; USDT parcial retém na conta)

---

### Estado: `FORWARDING`

**Causa provável:** rede congestionada, endereço SmartPay incorreto, saque bloqueado pela MEXC (allowlist).

**Diagnóstico:**
1. Pegar `withdrawalId` da trilha.
2. Na MEXC: `GET /capital/withdraw/history` filtrado pelo `withdrawalId`.
3. Status MEXC: `0=email_sent`, `1=cancelled`, `2=awaiting_approval`, `3=rejected`, `4=processing`, `5=failure`, `6=completed`.
4. Se status `4` (processing): aguardar até 2h antes de escalar.
5. Se status `5` ou `3`: falha confirmada.

**Decisão:**
- Endereço SmartPay correto mas saque falhou → `REVIEW_RETRY_FORWARD` após confirmar saldo USDT na conta master
- Endereço incorreto (errou ao chamar `withdrawOnchain`) → corrigir e `REVIEW_RETRY_FORWARD`
- USDT sumiu (status `6` mas SmartPay não recebeu) → escalar com suporte MEXC + SmartPay antes de qualquer decisão

---

### Estado: `PAYING_OUT`

**Causa provável:** chave PIX inválida/desativada, SmartPay com problema operacional, cliente com limite bancário.

**Diagnóstico:**
1. Pegar `payoutOrderId` da trilha.
2. Checar status do payout no dashboard/API SmartPay.
3. Verificar a chave PIX do cliente (`pixKey` no cash-out).

**Decisão:**
- Chave PIX inválida → contatar cliente, corrigir a chave → `REVIEW_RETRY_PAYOUT`
- SmartPay indisponível temporariamente → aguardar e `REVIEW_RETRY_PAYOUT`
- Chave definitivamente irrecuperável e cliente não responde → `REFUND_DECIDED` (USDT ainda na SmartPay; negociar devolução on-chain com a SmartPay)

---

### Estado: `MANUAL_REVIEW`

Estado de "sala de espera" — o sistema não conseguiu decidir sozinho. A trilha de eventos mostra o evento que trouxe para cá:

| Evento que causou | Situação | Ação recomendada |
|---|---|---|
| `DEPOSIT_MISMATCH` | Valor depositado diverge do cotado (< 99%) | Verificar diferença; se pequena → `REVIEW_RETRY_SELL` com o que chegou; se grande → `REFUND_DECIDED` |
| `SELL_UNDER_TOLERANCE` | Fill < 90% do esperado | Verificar USDT recebido; se suficiente → `REVIEW_RETRY_FORWARD`; senão → `REFUND_DECIDED` |
| `PAYOUT_FAILED` | SmartPay rejeitou o PIX | Ver estado `PAYING_OUT` acima |
| Reaper (stuck) | Timeout automático de estado intermediário | Diagnosticar pelo estado anterior ao `MANUAL_REVIEW` na trilha |

---

## 4. Critério objetivo: retomar vs. devolver vs. write-off

```
O ativo do cliente ainda está na nossa custódia?
├── SIM (conta master MEXC ou SmartPay) → retomar ou devolver
│   ├── Problema é temporário/corrigível → RETOMAR (REVIEW_RETRY_*)
│   └── Problema é permanente ou cliente prefere devolução → DEVOLVER (REFUND_DECIDED)
└── NÃO (ativo saiu mas destino não confirmou) → ESCALAR antes de qualquer ação
    └── Após investigação com MEXC/SmartPay sem resolução → WRITE_OFF + documentação obrigatória
```

**Regra inegociável:** `WRITE_OFF` só com evidência documentada de que o ativo foi perdido e não pode ser recuperado. Nunca por conveniência.

---

## 5. Como acionar as transições

```
POST /admin/cash-outs/:id/action
Authorization: Bearer <ADMIN_TOKEN>
Content-Type: application/json

{
  "action": "retry_sell",
  "reason": "MEXC voltou a negociar o par após suspensão de 20 min"
}
```

Valores válidos para `action`:

| action | Transição | Quando usar |
|---|---|---|
| `retry_sell` | MANUAL_REVIEW → SELLING | Refazer a venda spot |
| `retry_forward` | MANUAL_REVIEW → FORWARDING | Refazer o saque USDT → SmartPay |
| `retry_payout` | MANUAL_REVIEW → PAYING_OUT | Refazer o PIX (chave corrigida) |
| `refund` | MANUAL_REVIEW → REFUNDING_CRYPTO | Iniciar devolução da cripto |
| `write_off` | MANUAL_REVIEW → FAILED | Baixa terminal (irrecuperável) |
| `complete_refund` | REFUNDING_CRYPTO → REFUNDED | Confirmar devolução concluída |

O campo `reason` é **obrigatório** — fica nos logs para auditoria. Transição inválida retorna HTTP 422; cash-out inexistente retorna 404.

Qualquer ação que move dinheiro deve ser registrada com: ID da operação, quem decidiu, quando, por quê, resultado.

---

## 6. Pós-incidente

1. Confirmar estado terminal correto (`COMPLETED`, `REFUNDED` ou `FAILED`) via `GET /admin/cash-outs/:id`.
2. Verificar `GET /admin/cash-outs/:id/events` — a trilha deve ter o evento de resolução.
3. Registrar: causa-raiz, tempo de resolução, ação tomada.
4. Se o incidente se repetiu mais de uma vez: abrir ADR ou ajuste de timeout/retry.
5. Comunicar ao cliente se ele foi impactado (devolução, atraso no PIX).
