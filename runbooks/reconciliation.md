# Runbook: Reconciliação (cash-out)

> Como reconciliar o fluxo de **cash-out** (ADR-016) ao longo da cadeia: depósito
> do cliente → venda/saque (MEXC, convert) → payout em BRL (SmartPay).

> **STATUS: rascunho — a preencher**

## Pontos a casar (por operação)
- **Depósito do cliente** — cripto recebida (on-chain / `subaccount_balance` MEXC no
  convert; webhook SmartPay no FAST).
- **Venda → USDT** (convert) — ordem `sell_spot` na MEXC (`get_order`).
- **Forward** (convert) — saque do USDT para a SmartPay (`get_withdrawal`).
- **Payout** — PIX pago ao cliente (webhook/extrato SmartPay; `pix_e2e_id`).

## Fontes de verdade
- Depósito/venda/saque: MEXC (`get_deposit`, `get_order`, `get_withdrawal`, `subaccount_balance`). TODO: confirmar.
- Payout: webhook/extrato da SmartPay. TODO: confirmar.
- Estado interno: a máquina de estados (`../domain/state-machine.yaml`).

## Procedimento
1. TODO: casar cada cash-out (depósito ↔ venda ↔ forward ↔ payout ↔ estado).
2. TODO: conferir que todo `COMPLETED` tem PIX confirmado e todo `REFUNDED` tem a cripto devolvida.

## Alertas
- **Operações "recebido, não pago"** (`MANUAL_REVIEW`) sem dono → escalar.
- **Payout pendente** (`PAYING_OUT`) além do prazo → investigar com a SmartPay.

## Divergências e tratamento
- TODO. Atenção especial a `MANUAL_REVIEW` (cripto do cliente já recebida): cada
  caso precisa de resolução explícita (retomar venda/forward/payout, `REFUNDING_
  CRYPTO→REFUNDED`, ou `WRITE_OFF→FAILED`) — nada pode ficar pendente sem dono.
