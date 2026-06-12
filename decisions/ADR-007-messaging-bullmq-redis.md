# ADR-007 — Mensageria: cash-in assíncrono via fila (BullMQ/Redis)

> Como as etapas do cash-in são processadas e recuperadas de falha.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-12
- **Decisores:** Guilherme, CTO
- **Origem:** conselho do dev sênior (20 anos) — recomendou filas (Kafka ou
  RabbitMQ); o instinto é correto, a escolha de implementação para o MVP é do CTO.

## Contexto

Cash-in é intrinsecamente assíncrono: confirmar webhook da Eulen, executar a
compra spot na MEXC e disparar o saque on-chain são etapas que falham, demoram e
precisam de retry e idempotência. O dev sênior recomendou filas (Kafka ou
RabbitMQ). O instinto está certo; mas Kafka/RabbitMQ são infra pesada demais para
o MVP urgente, e a operação já usa Redis.

## Decisão

No MVP, o cash-in é processado de forma **assíncrona via fila**, usando
**BullMQ sobre Redis** (já em uso na operação). **Não** subir Kafka nem RabbitMQ
agora. A mensageria é uma **capability plugável (port)**: quando o volume
justificar, troca-se o adapter por Kafka/RabbitMQ **sem mexer na lógica de
negócio** (ADR-006).

## Consequências

- Cada etapa do cash-in (confirmar webhook, comprar spot, disparar saque) é um
  **job idempotente** com **retry/backoff** e **dead-letter**.
- A **chave de idempotência por etapa** precisa ser documentada
  (`domain/state-machine.yaml`). &lt;TODO: definir a chave de cada etapa.&gt;
- A máquina de estados referencia, por transição, qual job a dispara e o
  comportamento em falha (retry / dead-letter / estado de falha).
- BullMQ/Redis entra atrás de um `QueuePort`; o domínio não conhece BullMQ.

## Alternativas consideradas

- **Kafka** — recomendado pelo dev sênior; descartado no MVP (operação pesada,
  overkill para o volume atual). Candidato a adapter futuro.
- **RabbitMQ** — idem Kafka. Candidato a adapter futuro.
- **Processamento síncrono inline** — descartado: sem retry/idempotência
  robustos, inaceitável para movimento de dinheiro.
