# ADR-012 — Compliance: KYC/AML e gate de screening

> Onde a verificação de cliente e a prevenção à lavagem (PLD/AML) entram no fluxo,
> e o que isso obriga na arquitetura — mesmo com a POLÍTICA ainda em aberto.

> **STATUS: PROPOSTO — política pendente de reunião (sócios + especialista)**

- **Status:** Proposto
- **Data:** 2026-06-12
- **Decisores:** Guilherme, Mateus, Igor, CTO (+ assessoria regulatória — pendente)

## Contexto

O Lunium move **PIX → cripto** para usuários no Brasil **e como infraestrutura
para outros negócios (B2B)**. Isso ativa, no mínimo: regras de **PIX/Bacen**, o
regime **VASP** (Lei 14.478/2022 + regulação do Bacen como regulador de ativos
virtuais), obrigações de **PLD/AML** com reporte ao **COAF**, **KYC do usuário
final**, possível **travel-rule** no saque on-chain, e reporte fiscal (RFB). Como
a infra **toca o dinheiro e o PIX**, é plausível que a Lunium carregue o ônus
regulatório **independentemente** de quem "é dono" do cliente B2B.

Hoje isto é o **ponto cego declarado** do plano (`meetings/pauta-decisoes-pendentes.md`).
O risco de engenharia: finalizar a máquina de estados e o código **sem reservar o
ponto de entrada do screening** força um *retrofit* depois — exatamente num fluxo
financeiro que não deve ser remexido.

## Decisão

Esta decisão separa o que **já é arquitetural** do que **depende da reunião**:

**Arquitetural (decidido agora):**
- Reservar na máquina de estados um **gate de screening** como guard
  `screening_passed`, avaliado **antes de gerar o PIX** (transição
  `QUOTE_CREATED → PIX_PENDING`). Hoje é **no-op explícito** (seam), para que
  ativar a política seja configuração, não reescrita.
- O screening é uma **capability/port** (como os provedores): KYC, sanções/PEP e
  monitoramento de transação entram atrás de um port, com adapter trocável.
- **Trilha de auditoria** de cada decisão de screening (quem/quando/resultado),
  alinhada a `security/controls.md`.

**Pendente de reunião (NÃO decidido aqui — ver pauta):**
- **Quem faz o KYC do usuário final** — Lunium ou o cliente B2B (e como isso muda
  a responsabilidade regulatória).
- **Postura regulatória** — necessidade/Timing de registro VASP, enquadramento do
  arranjo PIX→DePix, obrigações de PLD/COAF, travel-rule no saque.
- **Provedor de screening** (KYC/sanções/monitoramento) e nível de exigência por
  ticket.

## Consequências

- A `domain/state-machine.yaml` ganha o guard `screening_passed` (seam reservado).
- Quando a política fechar, vira **ADR de aceite** (substitui o "Proposto") e o
  guard deixa de ser no-op; pode exigir um estado de **revisão de compliance**
  (ex.: pedido retido para análise) — encaixa no padrão de `MANUAL_REVIEW`.
- **Bloqueia a Fase 3** (primeiro dinheiro real): não se opera movendo dinheiro
  sem a postura de compliance definida. Resolver antes do go-live.

## Alternativas consideradas

- **Tratar compliance como fase 2** — descartado: KYC/AML não é feature, é
  pré-condição legal de operar; e enxertar o gate depois mexe no fluxo de dinheiro.
- **Assumir que o cliente B2B cobre todo o KYC** — não pode ser assumido sem
  parecer: quem toca o PIX/dinheiro tende a reter o ônus. Questão de reunião.
