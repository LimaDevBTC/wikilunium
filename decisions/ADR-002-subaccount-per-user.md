# ADR-002 — Sub-conta por usuário

> **⚠️ SUPERADO pelo [ADR-017](ADR-017-master-account-no-subaccounts.md)**
> (2026-06-22): limite de 30 sub-contas e exigência de API key por sub-conta
> inviabilizam o modelo para o MVP. Adotada conta master única + identificação
> por `txId`. Sub-contas retornam quando o MEXC Broker Program for aprovado.

- **Status:** Superado por ADR-017
- **Data:** 2026-06-10
- **Decisores:** Guilherme, Mateus, CTO

## Contexto (histórico)

Sem segregação por usuário, os fundos se misturam numa conta única (comingling) e
os depósitos ficam não-identificáveis. Precisamos de segregação nativa desde o início.

## Decisão original (revogada para MVP)

Sub-conta por usuário na MEXC com sub-contas virtuais, migrando para Broker Program.

## Por que foi superada

Ver ADR-017 para a análise completa. Em resumo:
- Limite duro de 30 sub-contas na conta standard.
- Cada sub-conta exige API key própria armazenada no banco (N pares de credenciais).
- Broker Program exige volume demonstrado — indisponível no lançamento.

## Decisão atual

Ver **ADR-017**: conta master única, identificação por `txId` on-chain.
