# Liquidity Model — Os Dois Mundos (define o escopo do MVP)

> **STATUS: rascunho — revisar com Guilherme**
> Este arquivo é a fonte de verdade sobre COMO o valor flui no cash-in e, por
> consequência, o que o MVP automatiza e o que fica de fora. `state-machine.yaml`,
> os providers e a reconciliação derivam daqui. Ver ADR-011.
> Nota de escopo: descreve a infra de forma client-agnostic. O comprecripto.io
> (Cliente #001) é o primeiro consumidor, mas o modelo vale para qualquer cliente.

## Princípio central

No MVP, o operador de liquidez (no Cliente #001, o Guilherme) atua como
**provedor de liquidez**. O DePix que entra pelo PIX do cliente final **NUNCA é
convertido na moeda final do cliente**. As duas pontas da operação são
desacopladas e mantidas por **estoques separados**.

O `lunium-api` automatiza **apenas o Mundo 1**.

-----

## Mundo 1 — Fluxo do cliente (AUTOMATIZADO pelo lunium-api)

1. Cliente final pede valor + moeda + carteira de destino (ex.: "R$100 em BTC").
2. **Pré-cheque de liquidez** — só cota/aceita se houver saldo no estoque de
   saída (MEXC) para entregar.
3. Geramos cobrança PIX via Eulen.
4. Cliente paga. Eulen credita → emite DePix (BRL 1:1 na Liquid) → DePix cai na
   carteira Liquid/SideSwap do operador de liquidez.
5. **A API da Eulen notifica o pagamento.** Este webhook é o GATILHO.
6. O gatilho dispara no estoque de saída (MEXC): compra spot da moeda pedida +
   saque on-chain para a carteira do cliente.
7. Cliente recebe a moeda. Operação concluída.

A carteira Liquid (entrada de DePix) e a carteira MEXC (saída de cripto) **não se
tocam em tempo real**. DePix entra de um lado; a moeda sai do outro, debitada de
um estoque pré-abastecido.

-----

## Mundo 2 — Reabastecimento de liquidez (MANUAL, FORA do sistema)

> `MVP: manual — fase 2: automatizar (ponte Liquid↔MEXC programática, rebalanceamento de estoque). Não confundir o estado atual com o design final.`

No MVP, o operador de liquidez faz isto por conta própria. O `lunium-api` **NÃO
executa nem orquestra** este mundo — apenas **observa** os saldos resultantes:

- Acumula DePix/BTC na Liquid (vindo do Mundo 1)
- Troca BTC → USDT na Liquid (SideSwap, manual)
- Faz a ponte de USDT Liquid → USDT/USDC em rede comum (Polygon / Ethereum /
  Solana), manual
- Deposita na MEXC para reabastecer o estoque de saída

Nosso sistema apenas **consome** o saldo da MEXC. Como esse saldo é reabastecido
é responsabilidade operacional do operador de liquidez.

-----

## Risco de primeira ordem: saldo do estoque de saída (Liquidity Failure)

Como os dois estoques são separados e reequilibrados manualmente, existe um risco
crítico: **o cliente paga o PIX, mas o estoque de saída (MEXC) não tem saldo (ou
não tem aquela moeda/rede) para entregar.** Recebemos o dinheiro e não
conseguimos cumprir.

Mitigações obrigatórias (entram na `state-machine.yaml` e em `security/`):

1. **Pré-cheque de saldo ANTES de cotar/aceitar.** Consultar saldo da MEXC na
   moeda+rede pedidas, contra o **disponível-para-prometer** (saldo − reservado em
   voo − margem), não o saldo bruto — senão, sob concorrência, vende-se o mesmo
   estoque a vários clientes (oversell). Mecanismo de reserva: **ADR-014**. Sem
   disponível suficiente → não oferecer / não gerar PIX.
2. **Estado explícito "pago mas não entregue"** (`AWAITING_LIQUIDITY` na
   `state-machine.yaml`). Auto-resume quando reabastecer (`LIQUIDITY_RESTOCKED`),
   ou reembolso ao cliente (`REFUNDING → REFUNDED`). Dinheiro do cliente nunca
   fica em limbo silencioso.
3. **Alerta de estoque baixo no dashboard** para o operador reabastecer ANTES de
   secar. Ele não pode descobrir que acabou quando um cliente já pagou.

-----

## Impacto na reconciliação

A reconciliação tem **dois eixos**, porque são dois estoques:

- **Entrada:** PIX recebido / DePix creditado (Eulen + carteira Liquid).
- **Saída:** moeda comprada + sacada (MEXC).

O dashboard mostra os dois lados e a saúde de cada estoque, com alerta para o
momento de reabastecer. Ver `../runbooks/reconciliation.md`.

-----

## Fronteira de responsabilidade (resumo)

| Etapa | Quem | Automatizado no MVP? |
|---|---|---|
| PIX → DePix → carteira Liquid | Eulen | Sim (observamos o webhook) |
| Webhook de pagamento → gatilho | lunium-api | Sim |
| Compra spot + saque on-chain | lunium-api | Sim |
| Reabastecer estoque de saída (Liquid→USDT→rede→MEXC) | Operador de liquidez | **Não — manual, fora do sistema (fase 2)** |

-----

## O que isto define sobre o escopo do MVP

**Entra:** cash-in determinístico ponta a ponta do Mundo 1, pré-cheque de
liquidez, dashboard de reconciliação de dois eixos, estados de falha com
resolução explícita.

**Fica para fase 2:** automação do Mundo 2 (reabastecimento), camada agêntica,
cash-out, múltiplos operadores/estoques simultâneos.
