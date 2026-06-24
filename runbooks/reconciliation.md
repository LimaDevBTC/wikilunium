# Runbook: Reconciliação (cash-out)

> Como verificar e reconciliar o fluxo de **cash-out** ao longo da cadeia:
> depósito do cliente → venda/saque MEXC (convert) → payout SmartPay (PIX).
>
> **STATUS: atualizado em 2026-06-23**

---

## 1. Quando rodar

- **Diariamente** (final do dia útil) — checar divergências acumuladas.
- **Após qualquer incidente** — confirmar que nenhuma operação ficou pendurada.
- **Antes de aumentar limites** (`DAILY_MAX_OPS` / `DAILY_MAX_BRL`) — garantir base limpa.

---

## 2. Passo a passo

### 2.1 — Snapshot do dia via API

```
GET /admin/reconciliation
Authorization: Bearer <ADMIN_TOKEN>
```

Retorna:
```json
{
  "cashoutEnabled": true,
  "todayUsage": { "ops": 12, "brl": "6480.00" },
  "divergences": [ ... ]
}
```

Se `divergences` está vazio e `cashoutEnabled` é `true`, o dia está limpo. Fim.

Se há divergências, cada item tem:
- `cashOutId` — ID da operação
- `state` — estado atual
- `stuckSinceMs` — há quanto tempo está naquele estado (ms)
- `reason` — descrição legível (ex.: `"parado em SELLING por 45 min (TTL: 30 min)"`)

---

### 2.2 — TTLs por estado (o que é normal vs. suspeito)

| Estado | TTL antes de ser divergência | Observação |
|---|---|---|
| `AWAITING_DEPOSIT` | `expiresAt` do quote (15 min) | Cliente não depositou no prazo |
| `DEPOSIT_DETECTED` | 30 min | Aguardando confirmações on-chain |
| `SELLING` | 30 min | Venda spot na MEXC travada |
| `SOLD` | 30 min | USDT na conta — forward não iniciou |
| `FORWARDING` | 60 min | Saque MEXC → SmartPay mais lento (normal até ~30 min) |
| `PAYING_OUT` | 30 min | SmartPay pagando PIX — pode travar com chave inválida |

---

### 2.3 — Investigar uma operação específica

```
GET /admin/cash-outs/:id/events
Authorization: Bearer <ADMIN_TOKEN>
```

Retorna a trilha ACID completa: cada evento com `from`, `to`, `event`, `idempotencyKey` e `createdAt`. Permite saber exatamente onde a operação parou e qual foi o último evento processado.

```
GET /admin/cash-outs?state=MANUAL_REVIEW&limit=50
Authorization: Bearer <ADMIN_TOKEN>
```

Lista todas as operações em `MANUAL_REVIEW` — estas exigem ação manual (ver §3).

---

### 2.4 — Conferir os terminais do dia

```
GET /admin/cash-outs?state=COMPLETED&limit=200
GET /admin/cash-outs?state=REFUNDED&limit=50
GET /admin/cash-outs?state=FAILED&limit=50
GET /admin/cash-outs?state=EXPIRED&limit=200
```

Verificar:
- Todo `COMPLETED` tem `payoutOrderId` preenchido (PIX confirmado pela SmartPay).
- Todo `REFUNDED` tem `withdrawalId` preenchido (saque de devolução concluído).
- Todo `FAILED` foi gerado por `WRITE_OFF` consciente (ver trilha de eventos).
- `EXPIRED` é esperado — cliente não depositou; nada foi movido.

---

## 3. O que fazer com cada tipo de divergência

### `AWAITING_DEPOSIT` expirado
O `expiresAt` passou e nenhum depósito chegou. O `ReaperWorker` já deve ter movido para `EXPIRED` automaticamente. Se ainda aparecer como `AWAITING_DEPOSIT` na reconciliação, o Reaper pode estar com problema — verificar logs do serviço.

**Ação:** confirmar que o Reaper está rodando; nenhuma intervenção manual necessária se o Reaper estiver saudável.

---

### `SELLING` parado > 30 min
A venda spot na MEXC pode estar travada (par suspenso, rede degradada, fill parcial abaixo da tolerância).

**Diagnóstico:**
1. Verificar `orderId` na trilha de eventos.
2. Checar o status da ordem diretamente na MEXC com o `orderId`.
3. Se a ordem foi cancelada ou rejeitada: mover para `MANUAL_REVIEW` e decidir retry.

**Ação (MANUAL_REVIEW):** ver §4.

---

### `FORWARDING` parado > 60 min
O saque USDT da MEXC para a SmartPay pode estar em fila de rede. Saques on-chain podem levar até 2h em congestão.

**Diagnóstico:**
1. Verificar `withdrawalId` na trilha de eventos.
2. Checar status do saque na MEXC (`GET /capital/withdraw/history`).
3. Se status = `processing` (2) ou `sent` (4): aguardar — não é incidente ainda.
4. Se status = `failed` (5) ou `rejected` (6): é incidente.

**Ação:** se falhou, ver runbook de incidente ([`incident-stuck-order.md`](incident-stuck-order.md)).

---

### `PAYING_OUT` parado > 30 min
SmartPay recebeu o USDT mas não confirmou o PIX. Causa mais comum: chave PIX inválida ou temporariamente indisponível.

**Diagnóstico:**
1. Verificar `payoutOrderId` na trilha.
2. Checar o status do payout na SmartPay (dashboard ou API).
3. Se a chave PIX é inválida: entrar em contato com o cliente para correção.

**Ação (MANUAL_REVIEW):** ver §4.

---

### `MANUAL_REVIEW` — qualquer
Cripto do cliente já está em nossa custódia (conta master MEXC ou SmartPay). **Cada caso precisa de resolução explícita.** Nada pode ficar em `MANUAL_REVIEW` indefinidamente.

Ver §4 abaixo.

---

## 4. Resolvendo `MANUAL_REVIEW`

Via `POST /admin/cash-outs/:id/action` com `ADMIN_TOKEN`:

```json
{ "action": "retry_sell", "reason": "MEXC voltou a negociar o par" }
```

| action | Quando usar |
|---|---|
| `retry_sell` | Falha temporária na MEXC; ativo ainda na conta master |
| `retry_forward` | Venda OK, saque falhou; USDT na conta master |
| `retry_payout` | USDT na SmartPay; PIX não confirmado |
| `refund` | Operação irrecuperável; cliente quer o ativo de volta |
| `complete_refund` | Após confirmar saque de devolução on-chain concluído |
| `write_off` | Cripto irrecuperável (documentar antes) |

O campo `reason` é **obrigatório** (auditoria em log). Retorna `{ cashOutId, action, state }`.

**Critério de devolução vs. write-off:**
- Se o ativo ainda está na conta master MEXC ou na SmartPay → devolução (`refund`).
- Se o ativo saiu e não pode ser recuperado → write-off (`write_off`) com documentação obrigatória.

---

## 5. Registro pós-reconciliação

Para cada divergência encontrada e resolvida, registrar:
- ID da operação
- Estado em que estava
- Causa identificada
- Ação tomada
- Resultado

Manter histórico para detectar padrões (ex.: MEXC degradada em horário específico, cliente recorrentemente enviando valor errado).
