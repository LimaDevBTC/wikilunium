# ADR-001 — Spot, não Convert

> Como o Lunium executa a compra de cripto na MEXC.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-10
- **Decisores:** Guilherme, Mateus, CTO

## Contexto

A compra de cripto na MEXC pode sair por duas rotas: **Convert** (conversão
direta, com spread embutido pelo provedor) ou **ordem spot** no livro (par contra
USDT). O ambiente anterior usava Convert e pagava o **spread opaco** embutido no
preço. Dois fatos fecham a decisão: (1) a **MEXC não expõe Convert na API** —
automatizar por Convert seria inviável; (2) o custo de spot é **transparente e
menor**: maker 0% / taker 0,05% contra um spread de Convert que não controlamos
nem medimos. Como movemos dinheiro de cliente, precisamos da rota mais barata,
**controlável e auditável** — e tratável em fill parcial.

## Decisão

A execução na MEXC é via **ordem spot** no par contra USDT. **No MVP (cash-out —
ADR-016) é a VENDA spot** do ativo depositado → USDT (caminho convert); no cash-in
(fase 2) seria a compra. Mesma lógica e os mesmos custos. O **fill parcial** é
tratado: a ordem "market" é modelada como **limit IOC** com tolerância de ~10%
(o IOC executa o que houver na hora e cancela o resto, limitando o slippage).

## Consequências

- Implica um **client spot tipado próprio** (a demo MEXC não serve em produção —
  Lunium.md §7), com assinatura HMAC e tratamento de erro/retry.
- **Tratamento de fill parcial** entra na máquina de estados: no cash-out (MVP),
  dentro da tolerância `SELLING → SOLD`; abaixo da tolerância `SELLING →
  MANUAL_REVIEW` (operador decide) — ver `domain/state-machine.yaml` e
  `domain/money-rules.md`.
- Exige **cálculo de preço/slippage** e validação de notional mínimo do par contra
  o catálogo (`exchange.get_catalog`).
- O risco de preço entre o quote e o fill fica com o operador durante o TTL do
  quote — TTL curto + markup cobrem isso (ver `money-rules.md` / pauta).

## Alternativas consideradas

- **Convert** — descartado: indisponível na API (mata a automação) e com spread
  mais caro e opaco que o spot.
- **Market order puro** — descartado: sem controle de slippage em livro fino;
  por isso a "market" é modelada como **limit IOC** com tolerância explícita.
