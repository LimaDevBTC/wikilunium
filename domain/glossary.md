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

- **On-ramp** — entrada de fiat → cripto (PIX → cripto). Provedor: Eulen.
- **Off-ramp** — saída cripto → fiat. **Fase 2** (SmartPay).
- **Cash-in** — fluxo completo fiat → cripto na carteira do usuário. **Escopo do
  MVP** (§4): Quote → PIX → compra spot → saque on-chain.
- **Cash-out** — fluxo inverso (cripto → fiat). **Fase 2**.
- **Quote** — cotação com valor, cripto, taxa e preço travados por um TTL.
  Primeiro estado do cash-in (`QUOTE_CREATED`).
- **Order** — a ordem de compra spot executada na exchange.
- **Withdrawal** — o saque on-chain da cripto para a carteira do usuário.

## Execução

- **Spot** — compra no mercado à vista da exchange (par contra USDT). Escolhido
  sobre o Convert. Ver ADR-001.
- **Fill parcial** — ordem executada só em parte. Tratada como limit **IOC** com
  tolerância de ~10%. Ver ADR-001.
- **IOC (Immediate-Or-Cancel)** — ordem que executa o que der na hora e cancela o
  resto.
- **Sub-conta** — conta segregada por usuário na exchange; dá segregação de
  fundos e depósito identificável. Ver ADR-002.

## Mensageria & estado

- **Máquina de estados** — os estados canônicos do cash-in, fonte única em
  [`state-machine.yaml`](state-machine.yaml).
- **Idempotência** — garantia de que repetir uma operação (ex.: webhook
  duplicado, retry de job) não causa efeito duplicado. Chave por etapa. Ver ADR-007.
- **Dead-letter (DLQ)** — destino de um job que esgotou os retries; isola a falha
  para inspeção. Ver ADR-007.
- **EXPIRED** — falha **antes** do pagamento: nenhum dinheiro movido, sem refund.
- **FAILED** — falha **depois** do pagamento: dinheiro retido, exige
  reconciliação/refund. Ver [`../runbooks/reconciliation.md`](../runbooks/reconciliation.md).

## Operação

- **Reconciliação** — conferência entre fiat (Eulen), ordens/saldos (MEXC) e
  saques on-chain. Base do dashboard do MVP (§4).
- **KYB (Know Your Business)** — verificação institucional da conta na exchange.
  Trava 1 conta por CNPJ; não converte PF→PJ depois. Ver §6.
