# Provedor: Eulen — On-ramp PIX

> Doutrina do provedor Eulen como capability `on_ramp` (entrada de fiat via PIX).
> Confirmado para o MVP.

> **STATUS: stub — a preencher**

<!-- Arquivo para HUMANO (doutrina). Os dados que o agente lê em runtime vivem em
     _capability-contract.yaml + config do provedor — não duplicar aqui. -->

## Papel (capability)

- Implementa: `on_ramp`

## Credenciais e ambiente

- TODO: confirmar com a conta Eulen empresarial **nova** (Lunium.md §6 — zero
  reaproveitamento do ambiente antigo).

## Endpoints

- TODO: confirmar endpoints (criar cobrança, consultar status).

## Webhook

- TODO: confirmar payload e verificação de assinatura do webhook de pagamento.

## Limites e rate limit

- Rate limit: ~10 ops/min — TODO: confirmar com a conta nova.

## Erros e normalização

- TODO: mapear erros Eulen → erros normalizados do domínio.

## Notas de integração / pegadinhas

- TODO.
