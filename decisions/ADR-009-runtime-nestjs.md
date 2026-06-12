# ADR-009 — Runtime e framework: TypeScript + Node.js + NestJS

> A linguagem, o runtime e o framework de aplicação do `lunium-api`.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-12
- **Decisores:** Guilherme, CTO

## Contexto

Faltava fechar o runtime/framework — o Lunium.md §3 marcava "a confirmar". As
decisões já tomadas restringem o espaço: a fila é **BullMQ** (ADR-007),
biblioteca do ecossistema Node; a arquitetura é **DDD + ports & adapters**
(ADR-006), que pede módulos com fronteiras claras e inversão de dependência. Sem
esta decisão não há como scaffoldar o `lunium-api`.

## Decisão

**TypeScript + Node.js**, com **NestJS** como framework de aplicação. Node casa
nativamente com BullMQ/Redis (ADR-007). NestJS entrega de fábrica módulos,
injeção de dependência e fronteiras explícitas, que mapeiam diretamente para
ports & adapters (ADR-006): domínio no centro; providers, banco e fila como
adapters atrás de ports. TypeScript tipa o domínio ponta a ponta (Quote, Order,
Withdrawal, contratos de provedor).

## Consequências

- Estrutura por **módulo de domínio** (Nest modules), não por camada técnica
  (ADR-006).
- **Ports = interfaces TS**; **adapters = providers concretos** injetados via DI.
- BullMQ via processadores/queues; cada etapa do cash-in é um job (ADR-007).
- Postgres atrás de um `PersistencePort`. &lt;TODO: escolher acesso a dados —
  Prisma vs TypeORM vs query builder — sem vazar para o domínio.&gt;
- Testes: unit no domínio (sem Nest), integração com fakes dos adapters (ADR-008).
- &lt;TODO: gestor de pacotes, versão LTS do Node, lint/format, pipeline de CI.&gt;

## Alternativas consideradas

- **TypeScript/Node + Fastify** — leve e sem opinião; exigiria montar DDD/DI na
  mão. Descartado por mais código de fundação e menos guard-rails de fronteira.
- **Outra linguagem (Go, etc.)** — descartado: BullMQ/Redis e o ecossistema já em
  uso são Node; trocaria reescreveria a mensageria sem ganho no MVP.
