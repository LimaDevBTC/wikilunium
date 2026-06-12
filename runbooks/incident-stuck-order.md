# Runbook: Incidente — ordem travada

> O que fazer quando uma ordem de cash-in trava (ex.: PIX pago mas compra/saque
> não concluiu) — resposta a incidente.

> **STATUS: stub — a preencher**

## Sintomas

- Operação parada além do prazo num estado intermediário (ex.: `PIX_PAID` sem
  virar `BUYING`, `WITHDRAWING` há horas). Normalmente o **reaper** (roadmap Fase 3)
  detecta e alerta antes do cliente.

## Diagnóstico

- Ver o **estado** na máquina (`../domain/state-machine.yaml`), os **logs** e o
  **WebSocket privado MEXC**. Distinguir:
  - **Espera legítima** (saque `processing`) → não é incidente; aguardar/re-checar.
  - **Sem estoque** → `AWAITING_LIQUIDITY` (reabastecer ou refund).
  - **Rede de saque suspensa** → `WITHDRAW_BLOCKED` (aguardar rede ou refund).
  - **Job esgotou retry / under-fill / saque travado no timeout** → `MANUAL_REVIEW`.

## Mitigação / resolução

- A partir de `MANUAL_REVIEW`, escolher a transição: **retomar** (`REVIEW_RETRY_BUY`
  / `REVIEW_RETRY_WITHDRAW`), **reembolsar** (`REFUND_DECIDED` → `REFUNDING`) ou
  **baixar** (`WRITE_OFF` → `FAILED`). Ação que move dinheiro exige confirmação +
  auditoria (ADR-010). Idempotência garante que reprocessar não duplica (ADR-007/013).
- TODO: critérios objetivos de quando retomar vs. reembolsar (depende de SLA/risco).

## Pós-incidente

- Reconciliar os dois eixos (ver [`reconciliation.md`](reconciliation.md)) e
  registrar a causa-raiz. Se recorrente, abrir ajuste de timeout/retry.
