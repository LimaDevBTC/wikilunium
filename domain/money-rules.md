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

## Representação monetária (tipo)

**Decidido:** todo valor monetário (BRL, USDT, cripto) é representado como
**decimal exato** — biblioteca de decimal ou inteiro em **unidades mínimas**
(satoshis, centavos, menor unidade do ativo). **Nunca** o `number` (float) do
JavaScript em valor de dinheiro: float introduz erro de arredondamento
inaceitável num sistema que move fundos. A serialização na API é **string**, não
número. Conversões e arredondamentos são explícitos, na fronteira, com a precisão
do ativo (abaixo).

## Arredondamento e precisão

**Decidido:** a precisão (casas decimais) de cada ativo **segue o catálogo da
MEXC** — o campo de precisão retornado por `exchange.get_catalog`
(`/api/v3/capital/config/getall`). Não há precisão fixa no Lunium; é **por ativo**.

**Decidido:** arredondar **truncando para baixo** a cripto entregue — nunca
entregar mais do que foi comprado (evita dust e saldo negativo na sub-conta). O
markup absorve a sobra de truncamento.

## Fill parcial

Tratado como **limit IOC com tolerância de ~10%** (ADR-001).

- **Dentro da tolerância** (`fill_within_tolerance`): a quantidade entregue é a
  `filled_qty` da ordem; transição `BUYING → BOUGHT`. O cliente recebe o
  efetivamente comprado (truncado para baixo); a diferença de preço/quantidade
  dentro da tolerância é absorvida pelo markup.
- **Abaixo da tolerância** (`not fill_within_tolerance`): transição
  `BUYING → MANUAL_REVIEW` (`domain/state-machine.yaml`). O dinheiro está retido
  e o operador decide: **entregar o parcial** (segue para `WITHDRAWING` com a
  quantidade ajustada), **recomprar o restante** (a preço novo — risco do
  operador) ou **reembolsar** (`REFUNDING`). Não há entrega silenciosa de valor
  divergente do cotado.

> TODO (parametrizar): o **valor exato da tolerância** (~10% é o teto do ADR-001)
> e a regra default de MANUAL_REVIEW por under-fill — depende do apetite de risco
> (ver pauta). O mecanismo já está na máquina; falta só o número/política.
