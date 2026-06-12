# ADR-013 — Outbox transacional (consistência fila ↔ banco)

> Como garantir que avançar o estado (Postgres) e enfileirar o próximo job (Redis)
> nunca divergem — sem ordem travada nem etapa duplicada.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-12
- **Decisores:** Guilherme, CTO

## Contexto

O cash-in encadeia transições de estado no **Postgres** (ADR-004, cada transição
em transação ACID) e jobs na fila **BullMQ/Redis** (ADR-007, cada etapa um job
idempotente). São **dois sistemas distintos**: se a aplicação commita a transição
no banco e **depois** enfileira o próximo job, uma falha entre os dois passos
deixa o estado avançado **sem job** (ordem travada) ou, se a ordem inverte, um job
disparado **sem o estado persistido** (execução fantasma/duplicada). É o clássico
problema de **dual-write** — inaceitável num sistema que move dinheiro e a causa
provável do incidente "ordem travada" (`runbooks/incident-stuck-order.md`).

Idempotência por etapa (ADR-007) protege contra *repetir* uma etapa, mas **não**
contra *perder* o agendamento da próxima.

## Decisão

Usar o padrão **transactional outbox**: o "próximo job a enfileirar" é gravado em
uma tabela **outbox no Postgres dentro da MESMA transação** que persiste a
transição de estado. Um **relay** (worker) lê a outbox e publica no BullMQ,
marcando como enviado de forma idempotente.

Garantias:
- Estado avançou ⇒ o job **está** na outbox (mesma transação) — nunca trava.
- Job só é publicado a partir de um estado **já commitado** — nunca fantasma.
- Republicação na falha do relay é segura: o consumidor é idempotente (ADR-007),
  então **at-least-once + idempotência = efeito exactly-once**.

## Consequências

- Tabela `outbox` + um **relay** publicando no BullMQ; a publicação é idempotente
  (dedupe por chave da etapa — `domain/state-machine.yaml`).
- O `cross_cutting.outbox` do `providers/_capability-contract.yaml` referencia
  este ADR.
- Pequeno custo de latência (o relay faz o despacho) — aceitável; cash-in já é
  assíncrono. <TODO: relay por polling vs. LISTEN/NOTIFY do Postgres.>

## Alternativas consideradas

- **Commit no banco e depois enfileirar (dual-write puro)** — descartado: janela
  de inconsistência = ordem travada ou job fantasma.
- **Two-phase commit entre Postgres e Redis** — descartado: complexidade alta,
  mau suporte, frágil. Outbox entrega a mesma garantia de forma mais simples.
- **Só idempotência (sem outbox)** — descartado: cobre duplicação, não cobre
  perda do agendamento da próxima etapa.
