# Glossário do domínio

> Vocabulário canônico do Lunium, para humanos — termos usados em specs, ADRs e
> código. Um termo, uma definição.

> **STATUS: rascunho — revisar**

## Arquitetura

- **Capability** — uma habilidade plugável que o Lunium consome (ex.: `on_ramp`,
  `exchange`). Implementada por um provedor. Princípio 1: provedores são
  ferramentas intercambiáveis.
- **Port** — a interface de uma capability (contrato), no centro do domínio.
  Definido em [`../providers/_capability-contract.yaml`](../providers/_capability-contract.yaml). Ver ADR-006.
- **Adapter** — a implementação concreta de um port por um provedor (ex.: Eulen
  é o adapter do port `on_ramp`; MEXC, do `exchange`). Ver ADR-006.
- **Cliente** — consumidor da infraestrutura Lunium (ex.: Cliente #001 =
  `comprecripto.io`). A infra é client-agnostic. Ver ADR-005.

## Fluxo de dinheiro

- **Off-ramp** — saída cripto → fiat (BRL no PIX). Provedor: SmartPay. **Escopo do
  MVP** (ADR-016). Dois caminhos: **modo FAST** e **modo convert** (ver abaixo).
  Ver `../providers/smartpay.md`.
- **Modo FAST** — cash-out instantâneo: o cliente envia **USDT em Polygon, Solana
  ou Tron** direto para a SmartPay, que paga o PIX em ~10s, sem conversão. Só para
  esses tokens/redes. Rápido, taxa maior.
- **Modo convert** — demais moedas → depósito na sub-conta MEXC → venda spot →
  USDT → saque p/ a SmartPay → PIX. Mais lento, taxa menor.
- **Cash-out** — fluxo completo cripto → fiat (cliente vende, recebe PIX).
  **Escopo do MVP** (§4 / ADR-016).
- **On-ramp** — entrada de fiat → cripto (PIX → cripto). Provedor: Eulen. **Fase 2.**
- **Cash-in** — fluxo fiat → cripto (cliente compra). **Fase 2** (ADR-016).
- **Quote** — cotação travada por um TTL. No cash-out: ativo, rede, quantidade,
  taxa, BRL a receber e caminho (FAST|convert). Primeiro estado (`QUOTE_CREATED`).
- **Order** — a ordem de **venda** spot executada na exchange (convert).
- **Withdrawal / forward** — saque on-chain do USDT para a SmartPay (ou devolução
  da cripto ao cliente, no refund).

## Execução

- **Spot** — negociação no mercado à vista da exchange (par contra USDT). No
  cash-out é **venda** (cripto → USDT). Escolhido sobre o Convert. Ver ADR-001.
- **Fill parcial** — ordem executada só em parte. Tratada como limit **IOC** com
  tolerância de ~10%. Ver ADR-001.
- **IOC (Immediate-Or-Cancel)** — ordem que executa o que der na hora e cancela o
  resto.
- **Sub-conta** — conta segregada por usuário na exchange; dá segregação de
  fundos e depósito identificável. Ver ADR-002.

## Mensageria & estado

- **Máquina de estados** — os estados canônicos do **cash-out** (ADR-016), fonte
  única em [`state-machine.yaml`](state-machine.yaml).
- **Idempotência** — garantia de que repetir uma operação (ex.: webhook
  duplicado, retry de job) não causa efeito duplicado. Chave por etapa. Ver ADR-007.
- **Outbox (transacional)** — o próximo job é gravado no Postgres na **mesma
  transação** da transição e publicado depois por um relay; evita estado avançado
  sem job (travado) ou job sem estado (fantasma). Ver ADR-013.
- **Dead-letter (DLQ)** — destino de um job que esgotou os retries; isola a falha
  para inspeção. Ver ADR-007.
- **Reaper** — vigia que detecta operações paradas além do prazo num estado e as
  escala (alerta + `MANUAL_REVIEW`). Ver `roadmap.md` Fase 3.

## Estados do cash-out (canônicos — `state-machine.yaml`)

- **QUOTE_CREATED → AWAITING_DEPOSIT** — cotação aceita; mostramos o endereço de
  depósito (SmartPay no FAST; sub-conta MEXC no convert) e aguardamos o cliente
  enviar a cripto.
- **DEPOSIT_DETECTED → DEPOSIT_CONFIRMED** — (convert) depósito do cliente visto e
  confirmado on-chain, batendo com a cotação.
- **SELLING → SOLD** — (convert) venda spot da cripto → USDT na MEXC.
- **FORWARDING** — (convert) saque do USDT para a SmartPay.
- **PAYING_OUT** — SmartPay recebeu o USDT e está pagando o PIX (FAST e convert
  convergem aqui).
- **MANUAL_REVIEW** — cripto do cliente já recebida e o fluxo automático não decide
  (depósito divergente, under-fill, payout falhou); operador resolve.
- **REFUNDING_CRYPTO → REFUNDED** — devolução da cripto ao cliente (caso "recebido,
  não pago").
- **COMPLETED** — cliente recebeu o PIX em BRL.
- **EXPIRED** — cliente não depositou no prazo: nada movido, falha limpa.
- **FAILED** — **baixa terminal (write-off)** irrecuperável e não reembolsável; só
  a partir de `MANUAL_REVIEW`. Exige reconciliação.

## Operação

- **Reconciliação** — conferência entre o depósito do cliente (on-chain/MEXC), a
  venda/saque (MEXC) e o payout em BRL (SmartPay). Base do dashboard do MVP (§4).
- **Reserva de liquidez (available-to-promise)** — *(cash-in / fase 2)* trava de
  estoque para evitar oversell. Não se aplica ao cash-out (capital-light). Ver ADR-014.
- **Screening** — checagem de compliance do cliente/operação (KYC, sanções/PEP,
  monitoramento) **antes do payout** (cripto→fiat é vetor de lavagem). Seam reservado
  na máquina (`screening_passed`); política pendente. Ver ADR-012.
- **KYB (Know Your Business)** — verificação institucional da conta na exchange.
  Trava 1 conta por CNPJ; não converte PF→PJ depois. Ver §6.
- **KYC (Know Your Customer)** — verificação do usuário final. Quem o executa
  (Lunium ou cliente B2B) é questão aberta. Ver ADR-012 / pauta.
- **IP de egress / allowlist** — provedores podem exigir que a API só aceite
  chamadas de um IP fixo cadastrado. O keyholder (`lunium-api`) tem **um** IP
  estático na allowlist; o front e os clientes não precisam de IP fixo. Ver ADR-015.
- **Keyholder** — o componente que detém as chaves de provedor e fala com eles
  (a `lunium-api`). Roda em host persistente; o front nunca é keyholder. Ver ADR-015.
