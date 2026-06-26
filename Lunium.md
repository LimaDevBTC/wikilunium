# Lunium — Brain Repository (wikilunium)

> **Este é o índice-mestre.** Claude Code e qualquer agente leem este arquivo
> PRIMEIRO. Ele aponta para toda a fonte de verdade do projeto.
> O código vive em `lunium-api` (cash-out headless) e `vendacripto-app`
> (front que consome a API). Aqui não roda nada — aqui se decide tudo.

-----

## 0. O que é o Lunium

**Infraestrutura headless de on/off-ramp cripto, AI-native** — o produto que o
Lunium vende. Clientes plugam nessa infra para oferecer compra/venda de cripto
aos seus usuários. O **Cliente #001 é `vendacripto.io`** (front próprio, repo
`vendacripto-app`, separado) — ver `clients/vendacripto.md` e o ADR-005.

A visão de produto é um **orquestrador de agentes** (compra e venda, texto/áudio),
com execução determinística. **No MVP a camada de agente está adiada** (ADR-003) e
o foco é **cash-out: o cliente VENDE cripto e recebe BRL no PIX** (ADR-016), via
SmartPay — caminho **FAST** (USDT em Polygon/Solana/Tron direto p/ a SmartPay, PIX
em ~10s) ou **convert** (demais moedas pela MEXC: venda spot → USDT → SmartPay).
**Cash-in (comprar cripto) é fase 2.** Tudo 100% determinístico.

**Princípios inegociáveis** (todo código e todo agente respeitam):

1. Provedores são ferramentas intercambiáveis. A lógica de negócio vive na
   camada Lunium, nunca acoplada a um provedor.
1. O usuário nunca custodia nada nosso e nós nunca custodiamos fiat. Orquestramos fluxo.
1. LLM aconselha e conduz; **execução de dinheiro é determinística**. Um agente
   nunca move fundos sozinho.
1. Segurança, risco e auditoria não são opcionais — são pré-condição de merge.
1. **Continuidade sem desenvolvedor**: se o CTO sair, o Guilherme continua. Este
   repo é o que garante isso. Tudo que está só na cabeça de alguém é um bug.

-----

## 1. Como usar este repo (método Karpathy)

- `wikilunium` (este repo): specs, ADRs, máquinas de estado, runbooks,
  doutrina de provedor, prompts. **Fonte de verdade.**
- [`lunium-api`](https://github.com/LimaDevBTC/lunium-api) (separado): serviço
  headless de **cash-out**. É o produto que o Lunium vende. Consome este brain para
  gerar e guiar código.
- `vendacripto-app` (separado): front Next.js que consome a `lunium-api` e
  serve no domínio vendacripto.io após o cutover de DNS.

Parte deste brain é **prosa para humanos** (`.md`) e parte é **estruturada para
máquina** (`.yaml`/`.json`) — porque os agentes parseiam estados, limites e
contratos de provedor em runtime. Não misturar: regra que o agente precisa ler
em runtime vai em arquivo estruturado.

**Regra de ouro de manutenção:** mudou uma decisão? Primeiro escreve o ADR aqui,
depois muda o código lá. Nunca o contrário.

-----

## 2. Mapa do repositório

```
wikilunium/
├── Lunium.md                  # você está aqui — índice-mestre
├── roadmap.md                 # plano de desenvolvimento da lunium-api (fases)
├── decisions/                 # ADRs (Architecture Decision Records)
│   ├── ADR-000-template.md
│   ├── ADR-001-spot-over-convert.md
│   ├── ADR-002-subaccount-per-user.md   # SUPERSEDED por ADR-017
│   ├── ADR-003-hybrid-llm-deterministic.md
│   ├── ADR-004-database-postgres.md       # banco principal: PostgreSQL
│   ├── ADR-005-platform-vs-client.md      # fronteira infra-vs-cliente
│   ├── ADR-006-ddd-ports-adapters.md      # arquitetura DDD + hexagonal
│   ├── ADR-007-messaging-bullmq-redis.md  # cash-in assíncrono via fila
│   ├── ADR-008-testing-strategy.md        # testes = pré-condição de merge
│   ├── ADR-009-runtime-nestjs.md          # runtime: TypeScript/Node + NestJS
│   ├── ADR-010-access-layers-channels.md  # camadas de acesso + bot operador
│   ├── ADR-011-liquidity-model.md         # dois mundos: World 1 auto, World 2 manual
│   ├── ADR-012-compliance-kyc-aml.md      # PROPOSTO: gate de screening (política pendente)
│   ├── ADR-013-transactional-outbox.md    # consistência fila↔banco (anti dual-write)
│   ├── ADR-014-liquidity-reservation.md   # reserva anti-oversell (available-to-promise)
│   ├── ADR-015-deployment-topology-egress-ip.md  # onde roda + IP de egress (lição btcnopix)
│   ├── ADR-016-mvp-pivot-cash-out.md      # MVP = cash-out (vender cripto → PIX); cash-in vira fase 2
│   └── ADR-017-master-account-no-subaccounts.md  # conta master + txId (supercede ADR-002)
├── domain/
│   ├── liquidity-model.md     # os dois mundos — define o escopo do MVP
│   ├── state-machine.yaml     # estados canônicos (máquina) — fonte única
│   ├── glossary.md            # vocabulário do domínio (humano)
│   └── money-rules.md         # ticket mínimo, taxa embutida, arredondamento
├── providers/                 # doutrina de cada provedor = "capability"
│   ├── _capability-contract.yaml   # interface que TODO provedor implementa
│   ├── smartpay.md            # off-ramp PIX — paga BRL (confirmado MVP — ADR-016)
│   ├── mexc.md                # exchange — recebe depósito + vende + saca p/ SmartPay (MVP)
│   ├── eulen.md               # on-ramp PIX-in (cash-in — FASE 2)
│   └── _candidates.md         # provedores citados, ainda não decididos
├── clients/                   # consumidores da infra Lunium
│   └── vendacripto.md        # Cliente #001 (front vendacripto-app, separado)
├── agents/                    # um arquivo por agente
│   ├── conversation-agent.md
│   ├── execution-agent.md
│   ├── monitor-agent.md
│   ├── risk-agent.md
│   └── asset-intelligence-agent.md
├── runbooks/                  # procedimentos operacionais (continuidade!)
│   ├── onboarding-mexc-broker.md
│   ├── key-rotation.md
│   ├── incident-stuck-order.md
│   └── reconciliation.md
├── security/
│   ├── threat-model.md        # inclui a lição do dev antigo
│   └── controls.md
└── meetings/
    ├── 2026-06-10-kickoff.md  # registro da call de origem (não é ata formal)
    └── pauta-decisoes-pendentes.md  # pontos em aberto p/ próxima reunião
```

-----

## 3. Estado atual (junho/2026)

|Item            |Status                                                                                                  |
|----------------|--------------------------------------------------------------------------------------------------------|
|Pessoas         |Guilherme (CEO/dono), Mateus (ops/corretoras), Igor (investidor USD 20k), CTO (você, USD 3k integrações)|
|Domínio         |`vendacripto.io` (registrado) — front do Cliente #001                                                                          |
|MVP             |**Cash-out ponta a ponta** (vender cripto → PIX via SmartPay) — FAST + convert (ADR-016)                |
|Modelo de agente|**Adiado** — MVP é cash-out 100% determinístico. Agêntico vira fase 2 (ADR-003)                         |
|Stack           |TypeScript/Node + NestJS (ADR-009) · Postgres (ADR-004) · DDD/hexagonal (ADR-006) · fila BullMQ/Redis (ADR-007) · VPS + IP de egress (ADR-015)|
|Off-ramp        |**SmartPay (PIX) — MVP**; recebe USDT e paga BRL                                                        |
|Exchange        |MEXC — recebe depósito, **vende** spot → USDT, saca p/ SmartPay (convert); **conta master + txId** (ADR-001, ADR-017)|
|`lunium-api`    |**Implementado** — camada de orquestração (Frente 1) + `MexcAdapter` real (Frente 2A) · [github.com/LimaDevBTC/lunium-api](https://github.com/LimaDevBTC/lunium-api)|
|On-ramp         |Eulen (PIX-in) — **fase 2** (cash-in)                                                                   |

-----

## 4. Escopo do MVP (corte fino)

**Entra (ADR-016):** **cash-out determinístico ponta a ponta** — o cliente vende
cripto e recebe BRL no PIX, via SmartPay, em dois caminhos:
- **FAST** — cliente envia **USDT (Polygon/Solana/Tron)** direto p/ a SmartPay →
  PIX em ~10s. Rápido, taxa maior. Sem MEXC.
- **Convert** — demais moedas → depósito na **conta master MEXC** (identificado por
  txId on-chain — ADR-017) → **venda spot → USDT** → saque do USDT p/ a SmartPay →
  PIX. Mais lento, taxa menor.

Inclui catálogo dinâmico da MEXC, **screening de compliance** antes do payout
(ADR-012) e reconciliação. Exposto como API headless para o `vendacripto-app`.

**Capital-light:** o cliente traz o ativo e a SmartPay frente o BRL — **sem estoque
pré-financiado nem reserva** (ADR-011/014 são de cash-in/fase 2).

**Fica para depois:** **cash-in (comprar cripto — Eulen/PIX-in)**, camada agêntica
(chat texto/áudio), múltiplos provedores simultâneos, troca automática de corretora,
PIX próprio, moeda própria.

A arquitetura **já nasce** preparada pra isso (capabilities plugáveis), mas o
código v1 é uma fatia fina e determinística.

-----

## 5. Decisões já fechadas (resumo — detalhe nos ADRs)

- **ADR-001 — Spot, não Convert.** MEXC não tem Convert na API. Ordem spot via
  par USDT é mais barata (maker 0% / taker 0.05%) que o spread do Convert que
  pagam hoje. Tratar fill parcial (market = limit IOC a 10%).
- **ADR-002 — Sub-conta por usuário. ⚠️ SUPERSEDED por ADR-017.** O limite de 30
  sub-contas e a necessidade de API key por sub-conta tornaram o modelo inviável.
- **ADR-017 — Conta master + txId.** Conta master única para todos os depósitos;
  identificação por txId on-chain. Idempotência via `newClientOrderId` (venda) e
  `withdrawOrderId` (saque). Caminho de migração para o Broker Program quando
  aprovado — o port `exchange` não muda, só o adapter.
- **ADR-003 — Agêntico adiado.** MVP é cash-in 100% determinístico. Quando a
  camada de agente entrar (fase 2), o modelo é híbrido: LLM conduz/aconselha,
  execução de dinheiro permanece determinística com confirmação explícita.
- **ADR-004 — Banco: PostgreSQL.** Domínio move dinheiro → consistência forte e
  ACID. Cada transição de estado roda em transação; nada de estado financeiro em
  store eventualmente-consistente.
- **ADR-005 — Plataforma vs cliente.** Lunium é infraestrutura headless
  multi-tenant-*ready*; cada consumidor é um cliente. Cliente #001 =
  `vendacripto.io` (front `vendacripto-app`, repo separado). Specs de infra
  ficam client-agnostic; multi-tenant nasce pronto na arquitetura, mas não é
  construído no MVP (um cliente só).
- **ADR-006 — DDD + Ports & Adapters.** Domínio isolado; provedores, banco e fila
  entram como adapters atrás de ports. Pastas por domínio, não por camada.
  Tradução em código do princípio "provedores são intercambiáveis".
- **ADR-007 — Mensageria (BullMQ/Redis).** Cash-in assíncrono via fila; cada
  etapa é job idempotente com retry/backoff e dead-letter. BullMQ/Redis no MVP
  (não Kafka/Rabbit); fila é port plugável, trocável depois sem tocar no domínio.
- **ADR-008 — Testes.** Unitários no domínio + integração nos fluxos de provedor
  = pré-condição de merge. Cobrir falhas: webhook duplicado, fill parcial, rede
  suspensa, saque que falha após a compra. Adapters têm fakes.
- **ADR-009 — Runtime: TypeScript/Node + NestJS.** Node casa com BullMQ (ADR-007);
  NestJS dá módulos/DI que mapeiam para ports & adapters (ADR-006). TS tipa o
  domínio ponta a ponta.
- **ADR-010 — Camadas de acesso e canais.** Três camadas (usuário/operador/
  suporte); brain só alimenta agente interno (externos usam KB curada). Operador
  (Guilherme) ganha bot determinístico de visibilidade (Telegram); versão agêntica
  é fase 2. Bot = mais um cliente da api headless.
- **ADR-011 — Modelo de liquidez (dois mundos).** Mundo 1 (entrega) automatizado
  pelo lunium-api via MEXC; Mundo 2 (reabastecer o estoque) manual no MVP, só
  observado. Pré-cheque de liquidez antes de cotar; reconciliação de dois eixos.
- **ADR-012 — Compliance/KYC-AML (PROPOSTO).** Reserva o gate de screening na
  máquina (`screening_passed`, hoje no-op); a **política** (quem faz KYC, VASP,
  PLD/COAF, travel-rule) é pendente de reunião e **bloqueia o go-live**.
- **ADR-013 — Outbox transacional.** O próximo job é gravado na mesma transação
  ACID da transição; elimina ordem travada / job fantasma do dual-write fila↔banco.
- **ADR-014 — Reserva de liquidez (anti-oversell).** O pré-cheque usa o
  disponível-para-prometer (saldo − reservado − margem), não o saldo bruto; evita
  vender o mesmo estoque a vários clientes. Parâmetros pendentes de reunião.
- **ADR-015 — Topologia de implantação e IP de egress.** O keyholder (`lunium-api`)
  roda em host persistente com **um IP estático** na allowlist dos provedores
  (~US$5/mês, não US$100 da Vercel — lição do btcnopix); o front (Vercel) não tem
  chave nem IP; nenhum cliente precisa de IP fixo. Segurança por camadas (chaves de
  privilégio mínimo + allowlist de saque + HMAC + cofre), não só IP.
- **ADR-016 — MVP pivota para cash-out.** O primeiro produto passa a ser **vender
  cripto → PIX via SmartPay** (FAST: USDT direto; convert: demais via MEXC). É
  capital-light (cliente traz o ativo; SmartPay frente o BRL). **Cash-in vira fase
  2**; Eulen sai do MVP; ADR-011/014 (estoque/reserva) são de cash-in.

-----

## 6. Caminho crítico (não é código — é burocracia)

Começa imediatamente, roda em paralelo ao código:

1. Conta **SmartPay** (off-ramp do MVP): credenciais, webhook de payout, redes/
   tokens aceitos, limites, prazo de liquidação, **pré-funding de BRL?** (ADR-016).
1. Conta **MEXC institucional (KYB)** no nome da entidade certa, com withdraw
   habilitado (saca USDT p/ a SmartPay) — ver `runbooks/onboarding-mexc-broker.md`.
   KYB trava 1 conta por CNPJ e não converte PF→PJ; nascer institucional desde já.
   **Nota ADR-017:** sem necessidade de sub-contas; conta master única suficiente.
1. Domínio + repos + chaves novas. **Zero reaproveitamento do ambiente antigo.**
1. (Fase 2) Conta Eulen empresarial — só quando o cash-in entrar.

-----

## 7. Referências técnicas externas

- MEXC API demo (oficial): endpoints, assinatura HMAC, sub-contas, WebSocket
  privado. Extrair specs daqui para `providers/mexc.md` e gerar client tipado
  próprio. Demo não é código de produção (sem retry/erro decente).
- Catálogo: `GET /api/v3/capital/config/getall` (min/max, taxa, redes por moeda).
- Pares negociáveis via API: filtrar via exchangeInfo / selfSymbol.
- Monitor em tempo real: WebSocket privado (PrivateOrders/PrivateDeals/
  PrivateAccount), polling só como fallback.

-----

## 8. Estado do código — `lunium-api` (2026-06-26)

Repositório: [github.com/LimaDevBTC/lunium-api](https://github.com/LimaDevBTC/lunium-api)
HEAD: `ce99648` · **134 testes passando**

### Concluído

**Frente 1 — Camada de orquestração (com fakes)**
- `PrismaPersistenceAdapter`: create / findById / applyTransition com outbox ACID (ADR-013)
- `OutboxRelay`: polling 500ms, publica `OutboxJob` no BullMQ por `idempotencyKey`
- `BullMqQueueAdapter`: `ConnectionOptions` via `REDIS_URL`
- `CashOutOrchestrator`: guards determinísticos, dispatch de effects por nome, `hasApplied` early-exit
- `QuoteService`: path FAST/DEX/CONVERT — DEX com cotação real-time via `DexSwapPort`, validação catálogo, `truncateDown`, TTL 15 min
- `CashOutController`: 4 endpoints (quote, accept, status, webhook SmartPay)
- `CashOutWorker` + `ReaperWorker` (expiração e stuck jobs)
- 20+ testes E2E com `FakePersistence` + `FakeQueue` (FAST, CONVERT, idempotência, falhas)

**Frente 2A — `MexcAdapter` real**
- `MexcHttpClient`: HMAC-SHA256, `X-MEXC-APIKEY`, parse de erros `code/msg`
- `MexcAdapter`: 8 métodos do `ExchangePort` (catálogo, depósito, venda IOC, saque, saldo)
- Normalização de redes (TRX↔tron, SOL↔solana, MATIC↔polygon)
- Idempotência no `sellSpot` (`DUPLICATE_CLIENT_ORDER` → busca por `origClientOrderId`)

**Frente 2B — `SmartPayAdapter` real**
- Autenticação JWT (`POST /v1/auth/login`) com cache 55 min + renovação automática
- `createPayoutOrder`: encode-memo via API (formato MemoDataEVM) + endereço SmartPay por rede (env)
- `getPayoutStatus`: `GET /v1/br/payments/off-ramp/status?txid=<txid>`
- `verifyWebhook`: valida `apiKey` no header `x-signature`, extrai `payoutOrderId`
- Fix orchestrator: CONVERT path obtém endereço SmartPay via `createPayoutOrder` antes do saque MEXC
- 20 testes unitários com `fetch` mockado

**Frente 2B-extra — testes unitários `MexcAdapter` (35 testes)**
- Cobre: getCatalog, getDepositAddress, getDeposit, sellSpot, getOrder, withdrawOnchain, getWithdrawal, getBalance
- Idempotência `DUPLICATE_CLIENT_ORDER`, normalização de erros, mapeamento de redes em ambas as direções

**Frente 3 — Operabilidade e limites**
- `DailyLimitService`: limites diários de nº de operações (`DAILY_MAX_OPS`) e volume BRL (`DAILY_MAX_BRL`)
- Circuit-breaker manual `CASHOUT_ENABLED=false` fecha todas as novas cotações (503)
- `AdminController`: `GET /admin/cash-outs` (filtro por estado, paginação), `GET /admin/cash-outs/:id/events` (trilha de auditoria), `GET /admin/reconciliation` — todos protegidos por `ADMIN_TOKEN`
- `ReconciliationService`: detecta cash-outs presos em estado intermediário além do TTL (AWAITING_DEPOSIT por `expiresAt`, demais por TTL fixo por estado)
- `POST /admin/cash-outs/:id/action`: 6 ações de resolução de MANUAL_REVIEW (retry_sell, retry_forward, retry_payout, refund, write_off, complete_refund); `reason` obrigatório

**Frente 4 — Hardening pré-go-live**
- `Dockerfile` multi-stage (node:20-alpine build + runtime) + `entrypoint.sh` que executa `prisma migrate deploy` antes de subir
- `ApiTokenGuard`: verifica `CLIENT_API_TOKEN` (Bearer) nos 3 endpoints públicos; webhook SmartPay e `/admin/*` isentos
- Rate limiting via `@nestjs/throttler`: 5 req/min em `POST /cash-outs`, 10 em `/accept`, 60 em `GET /:id`; override via `THROTTLE_LIMIT`
- `AlertService`: bot Telegram thin, fire-and-forget — 🚨 FAILED/MANUAL_REVIEW, ⚠️ REFUNDING_CRYPTO/reaper stuck, ℹ️ quotes expiradas; no-op sem credenciais (`TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID`)

**Frente 6 — Caminho DEX (quote-simulator do Luis)**
- `CashOutPath.DEX` — token não-USDT em Polygon/Solana/Tron → swap on-chain → USDT → SmartPay → PIX
- `DexSwapPort` + `DexSwapAdapter` (HTTP para o quote-simulator) + `FakeDexSwap`
- Orchestrator: effect `dexSwap.createSwapOrder` encadeia SmartPay (receiver) + DEX (HD wallet para o cliente)
- 8 novos testes de roteamento e fluxo E2E DEX (134 total)

### Pendente (Faixa B — aguarda credenciais reais)

- **Confirmar formato de webhook SmartPay** — verificar se o campo `reference` ecoa o cashOutId/memo (ver `providers/smartpay.md`)
- **Setup da conta SmartPay** — `POST /v1/application/create-source-address` para o endereço MEXC master; configurar webhook `PIX_OFFRAMP`
- **Deploy do quote-simulator** — configurar `DEX_SWAP_BASE_URL` + `DEX_SWAP_API_KEY` para ativar o caminho DEX
- **Testes de integração reais** — smoke test MEXC + SmartPay + DEX swap com valores mínimos
- **Período canário** — `CANARY_PIX_KEYS` configurado; reconciliação batendo 100% antes de abrir ao público

### Próximos passos técnicos

1. **Credenciais SmartPay** — configurar env vars, setup da conta (source-address + webhook), testar em sandbox
2. **Deploy do quote-simulator** — próprio repo, Docker, `DEX_SWAP_BASE_URL` configurado na lunium-api
3. **Testes de integração reais** — um ciclo completo FAST + DEX + CONVERT com valores mínimos
4. **Período canário** — `CANARY_PIX_KEYS` com chaves do time, N operações reais reconciliadas sem divergência