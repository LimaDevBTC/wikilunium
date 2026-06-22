# ADR-005 — Plataforma vs cliente (fronteira infra-vs-cliente)

> Quem é o produto e quem é o cliente — e o que isso proíbe vazar para os specs.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-12
- **Decisores:** Guilherme, CTO

## Contexto

O `vendacripto.io` é frequentemente confundido com o produto. Ele **não é**: é o
**primeiro cliente** da infraestrutura. Precisamos fixar essa fronteira para o
Lunium permanecer vendável a um Cliente #002 sem reescrita. Espelha o princípio
inegociável 1 (provedores são ferramentas intercambiáveis), aplicado agora
também aos **clientes**.

## Decisão

O **Lunium é infraestrutura headless multi-tenant-*ready***; cada consumidor é um
**cliente**. O **Cliente #001 é `vendacripto.io`** (front `vendacripto-app`,
repo separado — ver `clients/vendacripto.md`). Os specs de infra (`domain/`,
`providers/`, contratos) permanecem **client-agnostic**: nada específico de um
cliente vaza para eles. Multi-tenant **nasce pronto na arquitetura**, mas **não é
construído no MVP** (um cliente só).

## Consequências

- `clients/` registra cada cliente; a infra nunca referencia um cliente por nome.
- &lt;TODO: definir como o `lunium-api` identifica o tenant (chave/escopo) na fase
  de multi-cliente.&gt;

## Alternativas consideradas

- **Acoplar a infra ao comprecripto** (descartado — mata a revenda).
- **Construir multi-tenancy completo já no MVP** (descartado — over-engineering
  com um cliente).
