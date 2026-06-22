# Provedor: SmartPay — Off-ramp PIX (pagamento em BRL)

> Doutrina do provedor SmartPay como capability `off_ramp`: recebe **USDT** e
> **paga o cliente em BRL via PIX**. É a saída do cash-out.

> **STATUS: provedor CONFIRMADO do MVP (ADR-016) — stub de specs a preencher**
>
> O cash-out via SmartPay é o **fluxo do MVP** (ADR-016). A máquina de estados em
> `../domain/state-machine.yaml` é o cash-out. Faltam as specs reais da conta
> (endpoints, webhook, redes/limites) — Faixa B.

## Papel (capability)

- Implementa: `off_ramp` — entra cripto (USDT), sai **BRL no PIX** do cliente.

## Dois caminhos (cash-out)

- **Modo FAST** — o cliente envia **USDT em Polygon, Solana ou Tron** → vai
  **direto para a SmartPay** → a SmartPay paga o PIX em **~10s, tudo on-chain**.
  **Sem conversão, sem MEXC.** Vale **somente** para esses tokens/redes. É o
  caminho **rápido**, com **taxa maior** (premium pela velocidade — valor a definir).
- **Modo padrão (convert)** — "moedas normais": qualquer outra moeda → **entra na
  MEXC** → é **convertida em USDT** → o USDT é sacado (Polygon/Solana/Tron) **para a
  SmartPay** → a SmartPay paga o PIX. Tem etapa de conversão (preço + tempo): é
  **mais lento**, com **taxa menor**. Não é 10s.

O cliente escolhe o trade-off: **rápido e mais caro** (FAST) vs. **mais barato e com
espera** (convert).

A escolha do caminho é determinística pelo catálogo de saída:
`asset == USDT and network in {polygon, solana, tron}` → **FAST**; senão →
**convert** (reusa o adapter `exchange` da MEXC: venda spot + saque).

## Implicações / perguntas em aberto

- **Liquidez do off-ramp é da SmartPay** (ela paga o BRL ao receber o USDT) —
  modelo diferente do cash-in (onde o estoque é nosso, ADR-011). Confirmar se há
  pré-funding/limite e qual a exposição da Lunium.
- **Confiança/tempo no FAST:** a SmartPay paga ao detectar a entrada on-chain
  (Solana/Tron/Polygon finalizam em segundos). Confirmar a regra (mempool vs.
  confirmações) e o tratamento de **"USDT recebido mas PIX não pago"**.
- **Cash-out tem máquina de estados própria** (fase 2) — não confundir com o
  cash-in (`domain/state-machine.yaml`).

## Credenciais e ambiente · Endpoints · Webhook · Limites

- TODO (conta SmartPay): autenticação, endpoints, **webhook/confirmação de
  pagamento**, tokens/redes aceitos exatos, limites, prazo de liquidação,
  tratamento de falha. Egress por IP fixo da `lunium-api` se a SmartPay exigir
  allowlist (ADR-015).

## Erros e normalização

- TODO: mapear erros SmartPay → erros normalizados do domínio.
