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

- Implica **gestão de sub-contas** e um **mapeamento usuário↔sub-conta**
  persistido; `exchange.subaccount_create` é idempotente por `user_ref` e roda no
  **onboarding do usuário, antes do primeiro QUOTE** (a máquina de estados assume
  `subaccount_id` existente).
- **Reconciliação por sub-conta** (`exchange.subaccount_balance`), alinhada à
  reconciliação de dois eixos (ADR-011).
- **Caminho de migração para o Broker Program** quando aprovado — o port
  `exchange` não muda; troca-se o adapter/credencial.
- **Risco a verificar (Faixa B / pauta):** confirmar que sub-contas **virtuais**
  na API normal da MEXC suportam, no tier institucional e sem o Broker, depósito
  e saque independentes por sub-conta — e que o ToS permite segregar fundos de
  terceiros. É uma premissa load-bearing deste ADR.

## Alternativas consideradas

- **Conta única com memo/tag por depósito** — descartada: não segrega fundos
  (comingling), reconciliação e auditoria viram um pesadelo e some a fronteira
  por usuário.
- **Uma conta MEXC por usuário (não sub-conta)** — descartada: inviável
  operacionalmente (KYB por conta) e não escala.
