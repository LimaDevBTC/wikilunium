# Roadmap — desenvolvimento da lunium-api

> Plano de desenvolvimento do `lunium-api` (cash-in headless determinístico): em
> que ordem construir, do que cada fase depende e quando ela está "pronta". Não
> substitui os ADRs — **executa** o que eles decidiram.

> **STATUS: rascunho — revisar**

Ponto de entrada do projeto continua sendo o [`Lunium.md`](Lunium.md). Este plano
assume tudo que está fechado lá (§4 escopo; ADR-001…009). **Sem datas**: trata de
sequência e dependências; estimar com o time.

## Princípios que o plano respeita
- **Decisão primeiro, código depois.** Mudou algo? ADR aqui, depois código lá.
- **MVP é cash-in 100% determinístico** (ADR-003). Nada de camada agêntica.
- **Segurança/risco/auditoria são pré-condição de merge** (princípio 4 + ADR-008).

---

## Faixa B — Burocracia (começa JÁ, em paralelo — não é código, §6)
Bloqueia fases adiante; rodar desde o dia 1.
- [ ] Conta **Eulen** empresarial nova (credenciais, webhook, limites). → desbloqueia spec/adapter Eulen.
- [ ] Conta **MEXC** institucional (KYB no CNPJ certo, **withdraw habilitado**, sub-contas). → desbloqueia compra/saque reais. Ver [`runbooks/onboarding-mexc-broker.md`](runbooks/onboarding-mexc-broker.md).
- [ ] Domínio + repos + chaves novas. **Zero reaproveitamento do ambiente antigo.**

---

## Faixa A — Construção

### Fase 0 — Fechar o brain (pré-código)
**Objetivo:** eliminar os stubs que bloqueiam a geração de código; sair com a fonte de verdade preenchida.
**Depende de:** parcialmente da Faixa B (specs de provedor precisam das contas).
**Entregáveis:**
- [x] ADR-009 — runtime/framework (TypeScript/Node + NestJS).
- [ ] [`domain/state-machine.yaml`](domain/state-machine.yaml) — estados canônicos, transições, `failure_state`, `idempotency_key` por etapa.
- [ ] [`providers/_capability-contract.yaml`](providers/_capability-contract.yaml) — assinaturas (inputs/outputs), `failure_modes`, `limits` por port.
- [ ] [`providers/mexc.md`](providers/mexc.md) — endpoints reais, auth HMAC, limites, redes de saque (extrair da demo MEXC, §7).
- [ ] [`providers/eulen.md`](providers/eulen.md) — endpoints, webhook, limites (da conta nova).
- [~] [`domain/money-rules.md`](domain/money-rules.md) — arredondamento **decidido** (precisão por ativo da MEXC); **ticket mínimo e taxa dependem das integrações** (Faixa B).

**Pronto quando:** `state-machine.yaml` e `_capability-contract.yaml` sem TODO crítico; specs de provedor com endpoints/limites confirmados.

### Fase 1 — Scaffold da lunium-api (esqueleto DDD)
**Objetivo:** repositório de código vivo, vazio mas com a arquitetura certa.
**Depende de:** Fase 0 (contract + runtime).
**Entregáveis:**
- [ ] Repo `lunium-api`: NestJS + TypeScript, **estrutura por domínio** (não por camada) — ADR-006.
- [ ] **Ports** declarados como interfaces TS (ProviderPort on_ramp/exchange, PersistencePort, QueuePort).
- [ ] **PostgreSQL** + migrations + conexão (ADR-004), atrás do PersistencePort. &lt;decidir ORM — ADR-009.&gt;
- [ ] **BullMQ/Redis** ligado (ADR-007); um job dummy idempotente fim a fim.
- [ ] **Harness de testes** (unit + integração) + fakes base; **gate de CI** (ADR-008).

**Pronto quando:** build verde, CI rodando, job dummy idempotente passa em teste de integração.

### Fase 2 — Adapters de provedor (atrás dos ports)
**Objetivo:** Eulen e MEXC como adapters, testáveis sem bater na API real.
**Depende de:** Fase 1 + specs da Fase 0 + contas (Faixa B).
**Entregáveis:**
- [ ] Adapter **Eulen** (on_ramp): `create_pix_charge`, status, verificação de webhook. + **fake**.
- [ ] Adapter **MEXC** (exchange): catálogo (`getall`), `buy_spot` (spot, fill parcial IOC ~10% — ADR-001), `withdraw_onchain`, sub-contas (ADR-002), `get_order`, monitor via WebSocket privado. + **fake**.
- [ ] Erros normalizados (provedor → domínio); chamadas idempotentes.
- [ ] Integração cobrindo os cenários de falha do ADR-008: webhook duplicado, fill parcial, rede de saque suspensa, saque que falha após a compra.

**Pronto quando:** cada adapter satisfaz o contrato + fakes; cenários de falha verdes.

### Fase 3 — Orquestração do cash-in (máquina de estados sobre a fila)
**Objetivo:** o fluxo ponta a ponta, determinístico e recuperável.
**Depende de:** Fase 2 + MEXC com **withdraw habilitado** (Faixa B).
**Entregáveis:**
- [ ] Máquina de estados de domínio implementando [`state-machine.yaml`](domain/state-machine.yaml).
- [ ] Cada transição = **job BullMQ idempotente**, **1 transação ACID** (ADR-004/007), com retry/backoff, dead-letter e `failure_state`.
- [ ] Fluxo E2E: Quote → PIX QR Eulen → webhook → `buy_spot` MEXC → `withdraw` on-chain → carteira (sandbox/testnet).

**Pronto quando:** cash-in determinístico E2E verde em ambiente de teste; idempotência e recuperação de falha comprovadas.

### Fase 4 — Reconciliação + dashboard
**Objetivo:** fechar o MVP (§4).
**Depende de:** Fase 3.
**Entregáveis:**
- [ ] Reconciliação Eulen ↔ MEXC ↔ on-chain — preencher [`runbooks/reconciliation.md`](runbooks/reconciliation.md).
- [ ] **Dashboard de reconciliação**, exposto headless para o front consumir.

**Pronto quando:** divergências detectáveis e auditáveis; runbook de reconciliação completo.

### Fase 5 — Hardening, segurança e continuidade (pré-produção)
**Objetivo:** merge-ready e operável sem o autor original.
**Depende de:** Fase 3 (em paralelo com a 4).
**Entregáveis:**
- [ ] [`security/threat-model.md`](security/threat-model.md) + [`security/controls.md`](security/controls.md) preenchidos (incl. lição do dev antigo).
- [ ] Runbooks [`key-rotation.md`](runbooks/key-rotation.md) e [`incident-stuck-order.md`](runbooks/incident-stuck-order.md) preenchidos.
- [ ] Confirmação explícita de movimento de dinheiro; auditoria/logs.
- [ ] **Security review** aprovado (pré-condição de merge — princípio 4).

**Pronto quando:** security review passa; runbooks operacionais completos; checklist de go-live fechado.

---

## Integração com o front (`comprecripto-app`, Cliente #001 — ADR-005)
Paralelo a partir da Fase 2/3: estabilizar o **contrato da API headless** que o front consome; **cutover de DNS** para `comprecripto.io` no fim. Front é repo separado; a infra fica client-agnostic.

## Explicitamente FORA do MVP (não fazer agora)
Camada agêntica (fase 2 — ADR-003), cash-out / SmartPay, múltiplos provedores simultâneos, troca automática de corretora, PIX próprio, moeda própria. A arquitetura já nasce plugável para isso (capabilities/ports), mas o código v1 é fatia fina.

## Como manter este plano
Mudou escopo ou sequência? Atualize aqui **e** registre um ADR se for arquitetural. Marque as caixas conforme entrega; ao fechar uma fase, anote no [`CHANGELOG.md`](CHANGELOG.md).
