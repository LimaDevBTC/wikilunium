# Regras de dinheiro

> Ticket mínimo, taxa embutida, arredondamento e tratamento de fill parcial — as
> regras determinísticas que governam valores no **cash-out** (ADR-016).

> **STATUS: rascunho — revisar**

<!-- Arquivo para HUMANO. As regras viram código no domínio; os limites reais
     vêm dos provedores (catálogo MEXC + limites SmartPay). -->

## Ticket (mínimo e máximo)

**A definir — depende das integrações** (Faixa B do roadmap). O mínimo efetivo de
cash-out **não é um número escolhido**, e sim o **maior** entre os limites que se
aplicam na cadeia:

- mínimo de payout/recebimento da **SmartPay** (off-ramp);
- (convert) mínimo de ordem (notional) de **venda** na **MEXC** para o par;
- mínimo de depósito na rede escolhida (varia por rede).

A cotação (`QUOTE_CREATED`) valida o valor contra esses limites **antes** de aceitar,
e rejeita o que estiver abaixo do maior deles.

> TODO: confirmar cada limite com as contas reais (SmartPay + MEXC `get_catalog`) e
> derivar o ticket mínimo. Máximo: TODO (limites operacionais / risco).

## Taxa embutida (markup Lunium)

**A calcular.** O modelo (% sobre o valor, spread no preço, ou fixo + %) e o
número ainda serão definidos. No cash-out há **duas taxas** pela diferença de
custo/velocidade (ADR-016):

- **FAST** (USDT direto p/ a SmartPay) — mais rápido, **taxa maior** (premium).
- **Convert** (via MEXC) — mais lento, **taxa menor**; precisa cobrir o custo da
  venda spot + saque + o **risco de preço** entre cotar e vender.

> TODO: definir modelo e os dois números. A taxa fica **embutida no BRL da cotação**
> (transparente no Quote), não cobrada à parte. Ver `caderno`/pauta.

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

**Decidido:** arredondar **a favor do operador** — **truncar para baixo** o BRL
pago ao cliente (nunca pagar mais do que o USDT efetivamente rendeu/recebido) e a
precisão por ativo no convert. Evita pagar valor que não temos lastro; o markup
absorve a sobra de truncamento.

## Fill parcial (convert — venda spot)

Tratado como **limit IOC com tolerância de ~10%** (ADR-001).

- **Dentro da tolerância** (`fill_within_tolerance`): o BRL a pagar é derivado do
  `filled_qty` (USDT recebido) da venda; transição `SELLING → SOLD`. A diferença
  dentro da tolerância é absorvida pelo markup.
- **Abaixo da tolerância** (`not fill_within_tolerance`): transição
  `SELLING → MANUAL_REVIEW` (`domain/state-machine.yaml`). A cripto do cliente já
  está conosco e o operador decide: **pagar o proporcional ao que vendeu**,
  **vender o restante** (a preço novo — risco do operador) ou **devolver a cripto**
  (`REFUNDING_CRYPTO`). Não há payout silencioso divergente do cotado.

> TODO (parametrizar): o **valor exato da tolerância** (~10% é o teto do ADR-001)
> e a regra default de MANUAL_REVIEW por under-fill — depende do apetite de risco
> (ver pauta). O mecanismo já está na máquina; falta só o número/política.
