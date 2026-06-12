# Regras de dinheiro

> Ticket mínimo, taxa embutida, arredondamento e tratamento de fill parcial — as
> regras determinísticas que governam valores no cash-in.

> **STATUS: rascunho — revisar**

<!-- Arquivo para HUMANO. As regras viram código no domínio; os limites reais
     vêm dos provedores (catálogo MEXC + limites Eulen). -->

## Ticket (mínimo e máximo)

**A definir — depende das integrações** (Faixa B do roadmap). O mínimo efetivo de
cash-in **não é um número escolhido**, e sim o **maior** entre os limites que se
aplicam na cadeia:

- mínimo de cobrança PIX da **Eulen** (on-ramp);
- mínimo de ordem (notional) da **MEXC** para o par;
- mínimo de saque da **MEXC** na rede escolhida (varia por rede).

A cotação (`QUOTE_CREATED`) valida o valor contra esses limites **antes** de gerar
o PIX, e rejeita o que estiver abaixo do maior deles.

> TODO: confirmar cada limite com as contas reais (Eulen + MEXC `get_catalog`) e
> derivar o ticket mínimo. Máximo: TODO (limites operacionais / risco).

## Taxa embutida (markup Lunium)

**A calcular.** O modelo (% sobre o valor, spread no preço, ou fixo + %) e o
número ainda serão definidos.

> TODO: definir modelo e valor. A taxa deve ser **embutida no preço da cotação**
> (transparente no Quote), não cobrada à parte.

## Arredondamento e precisão

**Decidido:** a precisão (casas decimais) de cada ativo **segue o catálogo da
MEXC** — o campo de precisão retornado por `exchange.get_catalog`
(`/api/v3/capital/config/getall`). Não há precisão fixa no Lunium; é **por ativo**.

> Direção: recomendado **truncar para baixo** na cripto entregue (nunca entregar
> mais do que foi comprado; evita dust e saldo negativo). TODO: confirmar a direção.

## Fill parcial

Tratado como **limit IOC com tolerância de ~10%** (ADR-001). A quantidade
efetivamente entregue é a `filled_qty` da ordem (transição `BUYING → BOUGHT`),
dentro da tolerância.

> TODO: formalizar o tratamento quando o fill fica abaixo do cotado mas dentro da
> tolerância (ajuste da quantidade entregue vs. preço cobrado).
