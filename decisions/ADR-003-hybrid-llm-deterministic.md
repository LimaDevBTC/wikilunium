# ADR-003 — Agêntico adiado (híbrido LLM + determinístico)

> Onde entra (e onde NÃO entra) o LLM no Lunium.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-10
- **Decisores:** Guilherme, Mateus, CTO

## Contexto

A visão de produto é um **orquestrador de agentes** (o usuário fala "quero R$100
em BTC" e um agente conduz). Mas o MVP move dinheiro real de cliente e o tempo é
curto. Pôr um LLM no caminho de execução agora soma risco (não-determinismo,
prompt injection, alucinação) e superfície de ataque a um fluxo financeiro que
precisa ser auditável e reproduzível. O **princípio inegociável 3** já fixa:
LLM aconselha/conduz, **execução de dinheiro é determinística — um agente nunca
move fundos sozinho.** A decisão é sobre *quando* a camada agêntica entra.

## Decisão

O **MVP é cash-in 100% determinístico**; a camada de agente é **adiada para a
fase 2**. Quando entrar, o modelo é **híbrido**: o LLM **conduz e aconselha**,
mas a **execução de dinheiro permanece determinística** e exige **confirmação
explícita**. Um agente **nunca move fundos sozinho**.

## Consequências

- A pasta `agents/` nasce como **casca de fase 2**; o MVP **não pressupõe
  LLM/agente em runtime** — nenhum caminho de dinheiro depende de inferência.
- A `domain/state-machine.yaml` é 100% determinística: efeitos são `<port>.<method>`,
  guards são condições determinísticas, sem nó de decisão por LLM.
- Quando a camada entrar (fase 2), o modelo é **híbrido com confirmação
  explícita**: o agente propõe; a execução passa pelos mesmos jobs idempotentes e
  exige confirmação forte (espelha ADR-010 — ver vs. agir).
- O bot operador da Fase 4 é **determinístico** (menu/consulta), não agêntico —
  ter canal no Telegram não puxa o LLM (ADR-010).

## Alternativas consideradas

- **Agente executando dinheiro direto no MVP** — descartado: risco de
  não-determinismo/injection num fluxo financeiro, sem ganho que justifique agora.
- **Sem nenhum plano agêntico** — descartado: a arquitetura plugável já nasce
  pronta para a camada (capabilities/ports); só a *ativação* é adiada.
