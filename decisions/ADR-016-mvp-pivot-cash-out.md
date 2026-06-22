# ADR-016 — MVP passa a ser cash-out (vender cripto → PIX via SmartPay)

> Inverte a direção do primeiro produto: de **comprar** cripto (cash-in) para
> **vender** cripto e receber em BRL no PIX (cash-out).

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-21
- **Decisores:** Guilherme, CTO

## Contexto

O produto tem duas direções: **cash-in** (cliente paga PIX, recebe cripto) e
**cash-out** (cliente entrega cripto, recebe BRL no PIX). O brain foi todo modelado
para cash-in. Reavaliando a prioridade, o cash-out via **SmartPay** é melhor para
lançar primeiro:

- **Capital-light:** no cash-out **o cliente traz o ativo** e **a SmartPay frente o
  BRL** do PIX. Não precisamos pré-financiar um estoque de cripto nem manter float —
  o oposto do cash-in (que exige estoque na MEXC + reabastecimento manual; ADR-011).
- **Modo FAST é simples:** USDT em Polygon/Solana/Tron vai **direto para a SmartPay**,
  que paga o PIX em ~10s. Sem MEXC, sem conversão.
- Menos peças no caminho crítico → lançar mais rápido.

## Decisão

**O MVP passa a ser cash-out via SmartPay**, em dois caminhos
(ver `providers/smartpay.md`):

- **FAST** — cliente envia **USDT (Polygon/Solana/Tron)** → **direto para a
  SmartPay** → PIX em ~10s. Rápido, taxa maior.
- **Convert** — demais moedas → **entram na MEXC** (sub-conta do cliente) → **venda
  spot → USDT** → **saque do USDT para a SmartPay** → PIX. Mais lento, taxa menor.

**Cash-in (comprar cripto) vira fase 2.** A `domain/state-machine.yaml` é reescrita
para o fluxo de cash-out.

## Consequências

- **Providers:** **SmartPay (`off_ramp`) entra como provedor confirmado do MVP**;
  **Eulen (`on_ramp`, PIX-in) sai do MVP → fase 2**. A **MEXC** muda de papel: de
  comprar+sacar (cash-in) para **receber depósito + vender spot → USDT + sacar USDT
  para a SmartPay** (convert). FAST não toca a MEXC.
- **Some do MVP:** estoque pré-financiado e reconciliação de dois eixos como
  desenhados (ADR-011), reserva anti-oversell (ADR-014) e refund via PIX
  (`on_ramp.refund_pix`). São **específicos de cash-in → fase 2**.
- **Continuam válidos:** sub-conta por usuário (ADR-002 — agora para **depósito
  identificável do cliente** no convert), outbox (ADR-013), spot-não-Convert
  (ADR-001 — agora **venda** spot), runtime/banco/fila (ADR-004/007/009), topologia
  VPS + IP de egress (ADR-015), determinístico sem agente (ADR-003).
- **Compliance fica MAIS crítico:** cripto→fiat (pagar BRL) é vetor clássico de
  lavagem; o gate de screening (ADR-012) antes do payout é essencial.
- **Novo risco principal — o cliente move primeiro (irreversível):** precisamos
  tratar **depósito errado/atrasado/rede errada**, **risco de preço no convert**
  (entre cotar e vender) e **falha de payout da SmartPay**. Tudo modelado com
  estados de exceção (`MANUAL_REVIEW`, `REFUNDING_CRYPTO`).

## Perguntas em aberto (pauta)

- **A SmartPay frente o BRL** (paga ao receber o USDT) ou exige **pré-funding** de
  saldo BRL nosso? Define a exposição de capital.
- **Política de trava de preço (convert):** honramos a cotação se o depósito
  confirmar dentro do TTL, ou re-cotamos? Quem absorve o movimento no intervalo.
- **Orquestração do FAST:** endereço/tag de depósito da SmartPay por pedido, e o
  webhook de "recebido/pago"; nº de confirmações por rede.

## Alternativas consideradas

- **Manter cash-in primeiro** — descartado agora: exige capital de estoque e mais
  peças; cash-out lança mais rápido e leve.
- **Cash-in e cash-out juntos no MVP** — descartado: dobra o escopo do primeiro
  lançamento.
