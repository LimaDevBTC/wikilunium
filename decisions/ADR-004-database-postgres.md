# ADR-004 — Banco de dados: PostgreSQL

> O banco principal do `lunium-api`.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-12
- **Decisores:** Guilherme, CTO
- **Origem:** definição de arquitetura do CTO + conselho do dev sênior (20 anos)
  que revisou a infra ("trocar por Postgres/MySQL" — escolhido Postgres).

## Contexto

O domínio move dinheiro: cada cash-in encadeia PIX → compra spot → saque
on-chain, e a máquina de estados (`domain/state-machine.yaml`) precisa avançar de
forma atômica, sem estados financeiros inconsistentes. Um store
eventualmente-consistente abre janela para dupla execução, saldo fantasma e
transições parciais — inaceitável num sistema de pagamento.

Este ADR resolve a parte de **banco** do antigo ADR-004 (seleção de stack); a
escolha de runtime/framework permanece a confirmar (Lunium.md §3).

## Decisão

**PostgreSQL** como banco principal do `lunium-api`. Toda transição de estado
roda dentro de uma **transação ACID**; nada de estado financeiro em store
eventualmente-consistente. Consistência forte é requisito, não otimização.

## Consequências

- Cada transição de estado é commitada atomicamente (uma transação por
  transição); falha = rollback, sem estado intermediário persistido.
- Idempotência e travas (ex.: `SELECT ... FOR UPDATE`, constraints únicas) ficam
  no banco, alinhadas à mensageria por jobs (ADR-007).
- O acesso a dados entra como adapter atrás de um port de persistência (ADR-006)
  — o domínio não importa ORM/driver diretamente.
- &lt;TODO: versão mínima do Postgres, estratégia de migrations, host/gestão.&gt;

## Alternativas consideradas

- **MySQL** — também ACID; sugerido pelo dev sênior como alternativa. Descartado
  por preferência de ecossistema (JSONB, constraints, extensões). &lt;TODO: detalhar.&gt;
- **Store eventualmente-consistente (NoSQL/BaaS)** — descartado: incompatível com
  a exigência de atomicidade nas transições de dinheiro.
