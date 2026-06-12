# ADR-002 — Sub-conta por usuário

> Como o Lunium segrega fundos e identifica depósitos na MEXC.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-10
- **Decisores:** Guilherme, Mateus, CTO

## Contexto

Sem segregação por usuário, os fundos se misturam numa conta única (comingling) e
os depósitos ficam não-identificáveis — reconciliação e auditoria viram um
pesadelo. Precisamos de segregação nativa desde o início.

(A lição de *continuidade* do ambiente anterior é organizacional — concentração
de conhecimento, não comingling. Ver `security/threat-model.md`.)

## Decisão

Cada usuário recebe uma **sub-conta** na MEXC, dando **segregação nativa de
fundos** e **depósito identificável**. Começa com **sub-contas virtuais** (já
disponíveis na API normal) e **migra para o Broker Program** quando aprovado.
Evita comingling e garante depósito identificável desde o início.

## Consequências

&lt;TODO: implica gestão de sub-contas, mapeamento usuário↔sub-conta, caminho de
migração para Broker. Detalhar.&gt;

## Alternativas consideradas

&lt;TODO: conta única com memo/tag (descartada). Detalhar.&gt;
