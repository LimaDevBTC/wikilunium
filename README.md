# wikilunium — o brain do Lunium

Repositório-cérebro (wiki-LLM) do projeto **Lunium**, no método Karpathy: **a
fonte de verdade**. Aqui se decide tudo — specs, ADRs, máquinas de estado,
runbooks, doutrina de provedores. **Aqui não roda nada**: não há código de
aplicação neste repo.

## Ponto de entrada

Leia [`Lunium.md`](Lunium.md) **PRIMEIRO**. É o índice-mestre: princípios
inegociáveis, mapa do repositório, estado atual e decisões já fechadas. Qualquer
pessoa — ou qualquer agente — começa por ele.

## O que é o Lunium

Infraestrutura **headless** de on/off-ramp cripto. No MVP, faz **cash-in 100%
determinístico** ponta a ponta (PIX → compra spot → saque on-chain). É o produto
que o Lunium vende; clientes plugam nessa infra.

## Os três repositórios

- **wikilunium** (este) — o brain. Fonte de verdade. Não executa nada.
- **lunium-api** — serviço headless de cash-in. Consome este brain para gerar e
  guiar o código. **Repo separado.**
- **comprecripto-app** — front Next.js do **Cliente #001** (`comprecripto.io`),
  que consome a `lunium-api`. **Repo separado.** Ver
  [`clients/comprecripto.md`](clients/comprecripto.md).

> Lunium é a infraestrutura; `comprecripto.io` é um **cliente** dela — não o
> contrário. Os specs de infra ficam client-agnostic. Ver ADR-005.

## Como navegar

- [`decisions/`](decisions/) — ADRs (decisões de arquitetura). Comece pelo
  template `ADR-000`.
- [`domain/`](domain/) — máquina de estados (máquina), glossário e regras de
  dinheiro.
- [`providers/`](providers/) — doutrina de cada provedor + o contrato de
  capability (máquina).
- [`clients/`](clients/) — registro dos clientes da infra (#001 =
  `comprecripto.io`).
- [`agents/`](agents/) — **casca da fase 2** (camada agêntica adiada — não usar
  no MVP).
- [`runbooks/`](runbooks/) — procedimentos operacionais (continuidade).
- [`security/`](security/) — modelo de ameaças e controles.
- [`meetings/`](meetings/) — atas.

## Humano vs máquina

Arquivos `.md` são **prosa para humanos**. Arquivos `.yaml`/`.json` são
**estruturados**, lidos por geração de código e em runtime — ex.:
[`domain/state-machine.yaml`](domain/state-machine.yaml) e
[`providers/_capability-contract.yaml`](providers/_capability-contract.yaml).
Regra que o agente precisa ler em runtime vai em arquivo estruturado.

## Regra de ouro de manutenção

Mudou uma decisão? **Primeiro escreve o ADR aqui, depois muda o código no
`lunium-api`.** Nunca o contrário.

## Continuidade sem desenvolvedor

Este repo existe para que o projeto sobreviva à saída de qualquer pessoa — se o
CTO sair, alguém assume. **Se está só na cabeça de alguém, é um bug**: documente
aqui.
