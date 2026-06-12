# ADR-008 — Estratégia de testes

> O que é pré-condição de merge em termos de teste.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-12
- **Decisores:** Guilherme, CTO
- **Origem:** conselho do dev sênior (20 anos): "testes automatizados unitários e
  de integração".

## Contexto

É um sistema de pagamento; o princípio inegociável 4 já põe segurança/risco/
auditoria como pré-condição de merge. O risco real não está na aritmética do
domínio, e sim nos **fluxos de integração** com provedor, onde o mundo é hostil
(webhook duplicado, fill parcial, rede suspensa, saque que falha após a compra).

## Decisão

São **pré-condição de merge**: **testes unitários no domínio** + **testes de
integração nos fluxos de provedor**. Os **adapters de provedor têm fakes** para
teste sem bater na API real.

## Consequências

- Cobrir **explicitamente cenários de falha**, no mínimo:
  - webhook **duplicado** da Eulen (idempotência);
  - **fill parcial** na MEXC (ADR-001);
  - **rede de saque suspensa**;
  - **saque que falha após a compra** (estado intermediário + reconciliação).
- Cada adapter de provedor expõe um **fake** conforme o port (ADR-006).
- &lt;TODO: cobertura mínima, onde os fakes vivem, gate no CI.&gt;

## Alternativas consideradas

- **Só unitários** — descartado: não cobre o risco real (integração).
- **Só manual/E2E** — descartado: lento, não-determinístico, ruim como gate.
