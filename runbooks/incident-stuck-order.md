# Runbook: Incidente — ordem travada

> O que fazer quando uma ordem de cash-in trava (ex.: PIX pago mas compra/saque
> não concluiu) — resposta a incidente.

> **STATUS: stub — a preencher**

## Sintomas

- Operação de cash-out parada além do prazo num estado intermediário (ex.:
  `AWAITING_DEPOSIT` sem depósito, `PAYING_OUT` há tempo, venda/forward travados).
  Normalmente o **reaper** (roadmap Fase 3) detecta e alerta antes do cliente.

## Diagnóstico

- Ver o **estado** na máquina (`../domain/state-machine.yaml`), os **logs**, o
  **WebSocket MEXC** (depósito/venda) e o **webhook SmartPay** (payout). Distinguir:
  - **Cliente ainda não depositou** → `AWAITING_DEPOSIT` (aguarda; expira no prazo).
  - **Espera legítima** (saque/payout `processing`) → não é incidente; re-checar.
  - **Depósito divergente / under-fill na venda / payout falhou / job esgotou retry**
    → `MANUAL_REVIEW`.

## Mitigação / resolução

- A partir de `MANUAL_REVIEW`, escolher a transição: **retomar** (`REVIEW_RETRY_SELL`
  / `REVIEW_RETRY_FORWARD` / `REVIEW_RETRY_PAYOUT`), **devolver a cripto**
  (`REFUND_DECIDED` → `REFUNDING_CRYPTO`) ou **baixar** (`WRITE_OFF` → `FAILED`).
  Ação que move dinheiro exige confirmação + auditoria (ADR-010). Idempotência
  garante que reprocessar não duplica (ADR-007/013).
- TODO: critérios objetivos de quando retomar vs. devolver (depende de SLA/risco).

## Pós-incidente

- Reconciliar os dois eixos (ver [`reconciliation.md`](reconciliation.md)) e
  registrar a causa-raiz. Se recorrente, abrir ajuste de timeout/retry.
