# ADR-003 — Agêntico adiado (híbrido LLM + determinístico)

> Onde entra (e onde NÃO entra) o LLM no Lunium.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-10
- **Decisores:** Guilherme, Mateus, CTO

## Contexto

&lt;TODO: a preencher. Princípio inegociável: agente nunca move fundos sozinho.&gt;

## Decisão

O **MVP é cash-in 100% determinístico**; a camada de agente é **adiada para a
fase 2**. Quando entrar, o modelo é **híbrido**: o LLM **conduz e aconselha**,
mas a **execução de dinheiro permanece determinística** e exige **confirmação
explícita**. Um agente **nunca move fundos sozinho**.

## Consequências

&lt;TODO: a pasta `agents/` nasce como casca (fase 2). O MVP não pressupõe
agente/LLM em runtime. Detalhar.&gt;

## Alternativas consideradas

&lt;TODO: agente executando direto (descartado por risco). Detalhar.&gt;
