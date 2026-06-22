# Pauta — reunião de decisões pendentes

> Pontos em aberto que **não são decisão técnica** (essas viram ADR
> incrementalmente) e que **dependem dos sócios** — Guilherme, Mateus, Igor. Esta
> pauta sai dos TODOs do brain; quando a reunião acontecer, vira ata + ADRs.

> **STATUS: rascunho — pauta a discutir.**
> ⚠️ **O MVP pivotou para cash-out (ADR-016).** O documento de reunião atual e
> completo é o [`caderno-decisoes-reuniao.md`](caderno-decisoes-reuniao.md) (com PDF).
> Itens abaixo específicos de cash-in (oversell/float/reserva, refund via PIX, Eulen)
> são **fase 2**; o restante (compliance, precificação, entidade/KYB) segue valendo.

## Negócio
- **Precificação** — modelo da taxa embutida (% sobre o valor / spread no preço /
  fixo + %) e o número; ticket mínimo e máximo. → fecha
  [`../domain/money-rules.md`](../domain/money-rules.md) + possível ADR.
  ⚠️ **A taxa não pode ser fechada sem modelar o custo totalmente carregado das
  DUAS pontas**: MEXC taker 0,05% + taxa de saque on-chain por rede + custos do
  Mundo 2 (spread SideSwap + bridge + depósito) + **risco de preço no TTL do
  quote** (o operador absorve o movimento entre cotar e executar) + margem. Hoje
  os custos do Mundo 2 estão invisíveis na conta.
- **TTL do quote** — definir o prazo (curto, p/ limitar o risco de preço acima).
  → valor de `QUOTE_CREATED.timeout_s` na `state-machine.yaml`.
- **Orçamento / runway** — o que os aportes (Igor USD 20k; CTO USD 3k em
  integrações) cobrem e por quanto tempo. ⚠️ **O float do estoque de saída (MEXC)
  é capital de giro** e **capa o throughput do MVP**: com reabastecimento manual
  (Mundo 2), quanto volume diário USD ~20k sustenta, e qual a latência de restock?

## Operação / burocracia (Faixa B do roadmap)
- **Entidade do KYB** — em qual **CNPJ** abrir a conta MEXC institucional.
  **Irreversível** (1 conta por CNPJ; não converte PF→PJ depois).
- **Contas + prazos** — responsável e prazo para Eulen (empresarial) e MEXC
  (institucional, com withdraw + sub-contas).
- **Domínio** — confirmar `comprecripto.io` registrado + plano de cutover de DNS.

## Compliance (ponto cego nº 1 — ⚠️ BLOQUEIA O GO-LIVE) — ADR-012 (Proposto)
- **KYC / AML** — obrigações ao mover PIX→cripto no Brasil; **quem faz o KYC do
  usuário final** (Lunium ou o cliente B2B?); postura regulatória (Bacen/PIX,
  **registro VASP** — Lei 14.478/2022, **PLD/COAF**, **travel-rule** no saque
  on-chain, reporte fiscal). Cresce de importância porque a infra move dinheiro
  **para outros negócios**. → **ADR-012 já reservou o seam de screening na
  máquina**; falta a POLÍTICA. **Não é fase 2 — bloqueia a Fase 3 (dinheiro
  real).** Trazer assessoria regulatória.

## Liquidez, risco e provedores (da auditoria 2026-06-12)
- **Reserva de liquidez / oversell (ADR-014)** — definir a **margem de segurança**
  do pré-cheque e decidir: no volume do MVP, ligamos a reserva ou aceitamos
  oversell (contando só com `AWAITING_LIQUIDITY`)? Mecanismo já está pronto.
- **Tolerância de under-fill + política default** — valor da tolerância (~10% é o
  teto, ADR-001) e a regra quando o fill fica abaixo dela (entregar parcial vs.
  recomprar vs. refund — hoje cai em `MANUAL_REVIEW`).
- **MEXC — verificação e risco aceito** — confirmar que **sub-contas virtuais**
  suportam depósito/saque independentes e que o **ToS permite fundos de terceiros**
  (premissa do ADR-002). Aceitar conscientemente o **risco de concentração** (uma
  única exchange, sem fallback no MVP; risco de congelamento de conta)? Timeline do
  **Broker Program**.
- **Refund no MVP** — confirmar se a **Eulen** faz devolução de PIX via API
  (`refund_pix`) ou se o reembolso é **manual** (runbook); definir SLA de refund.
  Hoje o off-ramp é fase 2, então o caminho de reembolso precisa de dono.

## Como esta pauta vira decisão
Cada item resolvido na reunião → registra como **ADR** (se for
arquitetural/estrutural) ou preenche o doc correspondente (money-rules, runbooks).
A ata da reunião referencia os ADRs gerados.
