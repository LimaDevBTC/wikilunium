# Runbook: Reconciliação

> Como reconciliar os DOIS eixos do cash-in — entrada (PIX/DePix) e saída (MEXC) —
> já que são estoques separados (ADR-011 / `../domain/liquidity-model.md`).

> **STATUS: rascunho — a preencher**

## Os dois eixos
- **Entrada:** PIX recebido / DePix creditado — Eulen + carteira Liquid.
- **Saída:** moeda comprada + sacada — MEXC (estoque de saída).

Os dois estoques são desacoplados e reequilibrados manualmente (Mundo 2). A
reconciliação confere cada eixo e a saúde de cada estoque.

## Fontes de verdade
- Entrada: webhooks/extrato Eulen + saldo da carteira Liquid. TODO: confirmar.
- Saída: ordens + saques + saldo da MEXC (`subaccount_balance`). TODO: confirmar.
- Estado interno: a máquina de estados (`../domain/state-machine.yaml`).

## Procedimento
1. TODO: casar cada operação do Mundo 1 (entrada ↔ saída ↔ estado).
2. TODO: medir o saldo de cada estoque e a saúde (estoque de saída vs. demanda).

## Alertas
- **Estoque de saída baixo** → avisar o operador a reabastecer ANTES de secar (ele
  não pode descobrir que acabou quando o cliente já pagou).

## Divergências e tratamento
- TODO. Atenção especial aos estados de retenção (dinheiro do cliente parado):
  `AWAITING_LIQUIDITY`, `WITHDRAW_BLOCKED` e `MANUAL_REVIEW`. Cada um precisa de
  resolução explícita (retomar / `REFUNDING→REFUNDED` / `WRITE_OFF→FAILED`) — nada
  pode ficar pendente sem dono. Conferir também que cada `REFUNDED` **liberou** a
  reserva de liquidez (ADR-014).
