# Pauta — reunião de decisões pendentes

> Pontos em aberto que **não são decisão técnica** (essas viram ADR
> incrementalmente) e que **dependem dos sócios** — Guilherme, Mateus, Igor. Esta
> pauta sai dos TODOs do brain; quando a reunião acontecer, vira ata + ADRs.

> **STATUS: rascunho — pauta a discutir**

## Negócio
- **Precificação** — modelo da taxa embutida (% sobre o valor / spread no preço /
  fixo + %) e o número; ticket mínimo e máximo. → fecha
  [`../domain/money-rules.md`](../domain/money-rules.md) + possível ADR.
- **Orçamento / runway** — o que os aportes (Igor USD 20k; CTO USD 3k em
  integrações) cobrem e por quanto tempo.

## Operação / burocracia (Faixa B do roadmap)
- **Entidade do KYB** — em qual **CNPJ** abrir a conta MEXC institucional.
  **Irreversível** (1 conta por CNPJ; não converte PF→PJ depois).
- **Contas + prazos** — responsável e prazo para Eulen (empresarial) e MEXC
  (institucional, com withdraw + sub-contas).
- **Domínio** — confirmar `comprecripto.io` registrado + plano de cutover de DNS.

## Compliance (ponto cego atual — ⚠️)
- **KYC / AML** — obrigações ao mover PIX→cripto no Brasil; **quem faz o KYC do
  usuário final** (Lunium ou o cliente B2B?); postura regulatória (Bacen/PIX,
  VASP). Cresce de importância porque a infra move dinheiro **para outros
  negócios**. → provavelmente um ADR próprio.

## Como esta pauta vira decisão
Cada item resolvido na reunião → registra como **ADR** (se for
arquitetural/estrutural) ou preenche o doc correspondente (money-rules, runbooks).
A ata da reunião referencia os ADRs gerados.
