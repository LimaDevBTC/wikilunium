# Provedor: MEXC — Exchange (spot + saque)

> Doutrina do provedor MEXC como capability `exchange` (compra spot e saque
> on-chain). Confirmado para o MVP.

> **STATUS: stub — a preencher**

<!-- Arquivo para HUMANO (doutrina). Specs estruturadas → _capability-contract.yaml.
     Extrair detalhes da demo oficial MEXC (Lunium.md §7); a demo NÃO é código de
     produção (sem retry/erro decente) — gerar client tipado próprio. -->

## Papel (capability)

- Implementa: `exchange` (buy_spot, withdraw_onchain, catálogo, sub-contas)

## Credenciais e ambiente

- Conta **institucional (KYB)** no nome da entidade certa — ver
  `runbooks/onboarding-mexc-broker.md`.
- TODO: confirmar chaves e assinatura **HMAC** (Lunium.md §7).

## Endpoints relevantes

- Catálogo: `GET /api/v3/capital/config/getall` (min/max, taxa, redes por moeda).
- Pares negociáveis: filtrar via `exchangeInfo` / `selfSymbol`.
- TODO: confirmar endpoints de ordem spot, withdraw e sub-conta.

## Execução de compra

- Ordem **spot** contra USDT (ADR-001). Fill parcial: **limit IOC ~10%**.

## Sub-contas

- **Sub-conta por usuário** (ADR-002). Virtuais primeiro; Broker Program depois.

## Monitor (tempo real)

- **WebSocket privado** (PrivateOrders / PrivateDeals / PrivateAccount); polling
  só como fallback.

## Limites / taxas

- maker 0% / taker 0,05% (ADR-001). TODO: confirmar limites de saque por rede.

## Erros e normalização

- TODO: mapear erros MEXC → erros normalizados do domínio.
