# ADR-006 — Arquitetura: DDD + Ports & Adapters (hexagonal)

> Como o código do `lunium-api` é organizado.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-12
- **Decisores:** Guilherme, CTO
- **Origem:** definição de arquitetura do CTO + conselho do dev sênior (20 anos):
  "usar preferencialmente DDD".

## Contexto

Este brain já trata provedores como ferramentas intercambiáveis (princípio
inegociável 1) e mantém o domínio isolado (Quote, Order, Withdrawal, Provider,
máquina de estados, regras de dinheiro). Falta a tradução disso em código: se a
lógica de negócio importar SDK de provedor, ORM ou framework diretamente, a
promessa de intercambiabilidade morre na primeira integração.

## Decisão

O `lunium-api` segue **Domain-Driven Design** com **hexagonal / ports &
adapters**. O domínio fica no centro, isolado da infra. Provedores (Eulen, MEXC),
banco e mensageria entram como **adapters** atrás de **ports** (interfaces),
intercambiáveis sem tocar na lógica de negócio. É a tradução em código do
princípio "provedores são ferramentas intercambiáveis".

## Consequências

- **Estrutura de pastas por domínio**, não por camada técnica.
- A lógica de negócio **não importa** framework, ORM ou SDK de provedor
  diretamente — só ports.
- `providers/_capability-contract.yaml` é o contrato dos ports de provedor; cada
  `providers/<nome>.md` documenta um adapter.
- Persistência (ADR-004) e mensageria (ADR-007) também entram como adapters atrás
  de ports.
- &lt;TODO: nomear os ports do MVP — ProviderPort(on_ramp/exchange),
  PersistencePort, QueuePort — e onde vivem.&gt;

## Alternativas consideradas

- **Camadas técnicas (controller/service/repository)** — mais simples de começar,
  mas acopla domínio à infra e contraria o princípio 1. Descartada.
- **Acoplar direto aos SDKs dos provedores** — descartado: amarra a lógica a
  Eulen/MEXC e quebra a troca de adapter.
