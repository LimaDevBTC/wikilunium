# ADR-014 — Reserva de liquidez (anti-oversell)

> Como o pré-cheque de liquidez (ADR-011) evita vender o mesmo estoque a vários
> clientes ao mesmo tempo.

> **STATUS: rascunho — mecanismo decidido; parâmetros pendentes de reunião**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-12
- **Decisores:** Guilherme, CTO

## Contexto

O ADR-011 institui o **pré-cheque de liquidez**: só cotar/aceitar se houver saldo
no estoque de saída (MEXC). Mas um pré-cheque que **lê o saldo bruto** tem uma
falha de concorrência (TOCTOU): se N clientes cotam ao mesmo tempo, todos veem o
mesmo saldo, todos passam, todos pagam o PIX — e o estoque só cobre alguns. Os
demais caem em `AWAITING_LIQUIDITY` (pago, não entregue), segurando dinheiro de
cliente. É o clássico **oversell** de inventário. Com reabastecimento **manual**
(Mundo 2), o estoque demora a voltar — o impacto é real, não teórico.

## Decisão

O pré-cheque opera sobre o **disponível-para-prometer** (*available-to-promise*),
não sobre o saldo bruto:

```
disponível_para_prometer = saldo_MEXC(moeda, rede) − reservado_em_voo − margem
```

- **`QUOTE_ACCEPTED` reserva** a quantidade estimada (decrementa o disponível).
- **`EXPIRED` e `REFUNDED` liberam** a reserva.
- **`COMPLETED`** consome a reserva de forma definitiva (o saldo já saiu).
- O guard `liquidity_available` da `domain/state-machine.yaml` avalia o
  disponível-para-prometer, em transação (ADR-004), evitando corrida.

`AWAITING_LIQUIDITY` deixa de ser o caminho comum e volta a ser **exceção** (só
quando o estoque some por reabastecimento atrasado, não por contagem dupla).

## Consequências

- Estrutura de **reserva por moeda+rede** persistida (Postgres), atualizada nas
  transições de entrada/saída do funil.
- O guard lê reserva + saldo numa leitura consistente (`SELECT ... FOR UPDATE` ou
  equivalente — ADR-004).
- **Parâmetros pendentes (reunião):** o tamanho da **margem de segurança**, e se
  no volume do MVP se aceita rodar **sem reserva** (oversell tolerado, contando só
  com `AWAITING_LIQUIDITY`) para simplificar. O **mecanismo** fica pronto; ligar/
  ajustar é decisão de risco — ver `meetings/pauta-decisoes-pendentes.md`.

## Alternativas consideradas

- **Pré-cheque por saldo bruto (sem reserva)** — descartado como default:
  oversell sob concorrência. Pode ser aceito conscientemente no MVP de baixo
  volume (decisão de reunião), mas não é o desenho-alvo.
- **Reservar saldo na própria MEXC** (lock na exchange) — descartado: a MEXC não
  oferece hold de saldo para esse fim; a reserva é lógica, no nosso lado.
