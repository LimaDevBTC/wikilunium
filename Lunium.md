# Lunium — Brain Repository (wikilunium)

> **Este é o índice-mestre.** Claude Code e qualquer agente leem este arquivo
> PRIMEIRO. Ele aponta para toda a fonte de verdade do projeto.
> O código vive em `lunium-api` (cash-in headless) e `comprecripto-app`
> (front que consome a API). Aqui não roda nada — aqui se decide tudo.

-----

## 0. O que é o Lunium

**Infraestrutura headless de on/off-ramp cripto, AI-native** — o produto que o
Lunium vende. Clientes plugam nessa infra para oferecer compra/venda de cripto
aos seus usuários. O **Cliente #001 é `comprecripto.io`** (front próprio, repo
`comprecripto-app`, separado) — ver `clients/comprecripto.md` e o ADR-005.

A visão de produto é um **orquestrador de agentes**: o usuário fala (texto ou
áudio) “quero comprar R$100 em BTC”, um agente conduz a conversa, e a execução
determinística faz PIX → compra spot → saque on-chain para a carteira do
usuário. **No MVP a camada de agente está adiada** (ADR-003) — o cash-in é 100%
determinístico.

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
- `lunium-api` (separado): serviço headless de cash-in. É o produto que o
  Lunium vende. Consome este brain para gerar e guiar código.
- `comprecripto-app` (separado): front Next.js que consome a `lunium-api` e
  serve no domínio comprecripto.io após o cutover de DNS.

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
│   ├── ADR-002-subaccount-per-user.md
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
│   └── ADR-014-liquidity-reservation.md   # reserva anti-oversell (available-to-promise)
├── domain/
│   ├── liquidity-model.md     # os dois mundos — define o escopo do MVP
│   ├── state-machine.yaml     # estados canônicos (máquina) — fonte única
│   ├── glossary.md            # vocabulário do domínio (humano)
│   └── money-rules.md         # ticket mínimo, taxa embutida, arredondamento
├── providers/                 # doutrina de cada provedor = "capability"
│   ├── _capability-contract.yaml   # interface que TODO provedor implementa
│   ├── eulen.md               # on-ramp PIX (confirmado MVP)
│   ├── mexc.md                # exchange (confirmado MVP)
│   ├── smartpay.md            # off-ramp PIX (fase 2)
│   └── _candidates.md         # provedores citados, ainda não decididos
├── clients/                   # consumidores da infra Lunium
│   └── comprecripto.md        # Cliente #001 (front comprecripto-app, separado)
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
|Domínio         |`comprecripto.io` (registrado) — front do Cliente #001                                                                          |
|MVP             |**Cash-in ponta a ponta + dashboard de reconciliação**                                                  |
|Modelo de agente|**Adiado** — MVP é cash-in 100% determinístico. Agêntico vira fase 2 (ADR-003)                          |
|Stack           |TypeScript/Node + NestJS (ADR-009) · Postgres (ADR-004) · DDD/hexagonal (ADR-006) · fila BullMQ/Redis (ADR-007)|
|Exchange        |MEXC via spot API + sub-conta por usuário (ADR-001, ADR-002)                                            |
|On-ramp         |Eulen (PIX)                                                                                             |
|Off-ramp        |SmartPay (fase 2)                                                                                       |

-----

## 4. Escopo do MVP (corte fino)

**Entra:** cash-in determinístico ponta a ponta (Quote → PIX QR Eulen → webhook
→ spot buy MEXC → withdraw on-chain → carteira do usuário), **pré-cheque de
liquidez** antes de cotar, catálogo dinâmico da MEXC, **dashboard de reconciliação
de dois eixos**. Exposto como API headless para o `comprecripto-app` consumir.
Cliente principal: comprecripto.io.

O operador atua como **provedor de liquidez**: a entrega sai de um **estoque MEXC
pré-financiado**, desacoplado da entrada de DePix; reabastecer esse estoque é
**manual** no MVP (Mundo 2). Ver `domain/liquidity-model.md` + ADR-011.

**Fica para depois:** camada agêntica (chat texto/áudio), cash-out, múltiplos
provedores simultâneos, troca automática de corretora, PIX próprio, moeda própria.

A arquitetura **já nasce** preparada pra isso (capabilities plugáveis), mas o
código v1 é uma fatia fina e determinística.

-----

## 5. Decisões já fechadas (resumo — detalhe nos ADRs)

- **ADR-001 — Spot, não Convert.** MEXC não tem Convert na API. Ordem spot via
  par USDT é mais barata (maker 0% / taker 0.05%) que o spread do Convert que
  pagam hoje. Tratar fill parcial (market = limit IOC a 10%).
- **ADR-002 — Sub-conta por usuário.** Segregação nativa de fundos e depósito
  identificável. Começa com sub-contas virtuais (já na API normal), migra para
  Broker Program quando aprovado. Evita comingling de fundos.
- **ADR-003 — Agêntico adiado.** MVP é cash-in 100% determinístico. Quando a
  camada de agente entrar (fase 2), o modelo é híbrido: LLM conduz/aconselha,
  execução de dinheiro permanece determinística com confirmação explícita.
- **ADR-004 — Banco: PostgreSQL.** Domínio move dinheiro → consistência forte e
  ACID. Cada transição de estado roda em transação; nada de estado financeiro em
  store eventualmente-consistente.
- **ADR-005 — Plataforma vs cliente.** Lunium é infraestrutura headless
  multi-tenant-*ready*; cada consumidor é um cliente. Cliente #001 =
  `comprecripto.io` (front `comprecripto-app`, repo separado). Specs de infra
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

-----

## 6. Caminho crítico (não é código — é burocracia)

Começa imediatamente, roda em paralelo ao código:

1. Conta Eulen empresarial nova (credenciais, webhook, limites — rate limit 10 ops/min)
1. Conta MEXC institucional (KYB) no nome da entidade certa, com withdraw
   habilitado — ver `runbooks/onboarding-mexc-broker.md`. KYB trava 1 conta por
   CNPJ e não converte conta PF→PJ depois; nascer institucional desde já.
1. Domínio + repos + chaves novas. **Zero reaproveitamento do ambiente antigo.**

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

## 8. Próximos passos imediatos

1. Criar este repo `wikilunium` com a estrutura da seção 2.
1. Preencher `meetings/2026-06-10-kickoff.md` com a ata (já temos o conteúdo).
1. Escrever ADR-001, 002, 003 (decisões fechadas) e abrir ADR-004 (stack).
1. Definir `domain/state-machine.yaml` e `providers/_capability-contract.yaml` —
   são os dois arquivos que destravam a geração de código.
1. Só então abrir `lunium-api` e gerar o esqueleto a partir do brain.