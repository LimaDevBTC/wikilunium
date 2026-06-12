# ADR-011 — Modelo de liquidez (dois mundos)

> Como o valor flui no cash-in: o operador como provedor de liquidez, com
> estoques de entrada e saída desacoplados.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-12
- **Decisores:** Guilherme, CTO

## Contexto

O cash-in **não** é um repasse em tempo real: o DePix que entra pelo PIX não é
convertido na moeda final do cliente. O operador de liquidez (no Cliente #001, o
Guilherme) mantém **dois estoques separados**, reequilibrados manualmente.
Precisamos fixar o que o `lunium-api` automatiza, o que fica de fora, e o risco
que isso cria. Detalhe completo em `domain/liquidity-model.md`.

## Decisão

**Dois mundos:**
- **Mundo 1 — fluxo do cliente (AUTOMATIZADO pelo lunium-api):** a Eulen notifica
  o pagamento (webhook = gatilho); o **lunium-api compra na MEXC (estoque de saída
  pré-financiado) e saca on-chain** para a carteira do cliente. Entrada (DePix na
  Liquid) e saída (MEXC) **não se tocam em tempo real**.
- **Mundo 2 — reabastecimento (MANUAL no MVP; observado, não orquestrado):** o
  operador converte o que recebe (DePix/Liquid) e deposita na MEXC para
  reabastecer o estoque de saída. O `lunium-api` **apenas observa** o saldo
  resultante. Automatizar isso é fase 2.

**Pré-cheque de liquidez:** só cota/aceita/gera PIX se houver **disponível-para-
prometer** no estoque de saída (MEXC) na moeda+rede pedidas (+ margem). A reserva
que evita oversell sob concorrência é o **ADR-014**. Sem disponível → não oferece.

**Falha de liquidez ("pago, não entregue"):** se, apesar do pré-cheque, faltar
estoque na hora de entregar, há um estado explícito de retenção
(`AWAITING_LIQUIDITY`) — auto-resume ao reabastecer (`LIQUIDITY_RESTOCKED`), ou
reembolso (`REFUNDING → REFUNDED`). Dinheiro do cliente nunca em limbo silencioso.
(Modelagem exata dos estados: `domain/state-machine.yaml`.)

**Reconciliação de dois eixos:** entrada (PIX/DePix — Eulen + Liquid) e saída
(compra+saque — MEXC), com saúde de cada estoque e alerta de estoque baixo.

## Consequências

- O `lunium-api` precisa do **pré-cheque de saldo** (port `exchange`:
  `subaccount_balance` + redes do catálogo) antes de cotar.
- A `state-machine.yaml` ganha o **estado de retenção por liquidez** e o guard de
  pré-cheque.
- Reconciliação e dashboard são de **dois eixos** (`runbooks/reconciliation.md`).
- O **float do estoque de saída é capital de giro do operador** — item de
  runway/orçamento (`meetings/pauta-decisoes-pendentes.md`).

## Alternativas consideradas

- **Repasse em tempo real** (converter o DePix recebido direto na moeda do
  cliente) — descartado: acoplaria as pontas e dependeria de liquidez instantânea
  por operação; estoque pré-financiado é mais robusto.
- **Automatizar o Mundo 2 já no MVP** — descartado: ponte Liquid→MEXC programática
  é fase 2; o MVP entrega cash-in rápido com reabastecimento manual.
