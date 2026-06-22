# Provedor: MEXC — Exchange (spot + saque)

> Doutrina do provedor MEXC como capability `exchange` (compra spot e saque
> on-chain). Confirmado para o MVP.

> **STATUS: stub — a preencher**

<!-- Arquivo para HUMANO (doutrina). Specs estruturadas → _capability-contract.yaml.
     Extrair detalhes da demo oficial MEXC (Lunium.md §7); a demo NÃO é código de
     produção (sem retry/erro decente) — gerar client tipado próprio. -->

## Papel (capability) — cash-out (ADR-016)

- Implementa: `exchange`. No cash-out (caminho **convert**): **recebe o depósito**
  do cliente na sub-conta (depósito identificável — ADR-002), **vende spot →
  USDT** (`sell_spot`) e **saca o USDT para a SmartPay** (`withdraw_onchain`).
  Também: `get_deposit_address`, `get_deposit`, catálogo, sub-contas.
- O caminho **FAST** (USDT direto p/ a SmartPay) **não toca a MEXC**.
- (Cash-in/`buy_spot` é fase 2.)

## Credenciais e ambiente

- Conta **institucional (KYB)** no nome da entidade certa — ver
  `runbooks/onboarding-mexc-broker.md`.
- TODO: confirmar chaves e assinatura **HMAC** (Lunium.md §7).

## Endpoints relevantes

- Catálogo: `GET /api/v3/capital/config/getall` (min/max, taxa, redes por moeda).
- Pares negociáveis: filtrar via `exchangeInfo` / `selfSymbol`.
- TODO: confirmar endpoints de ordem spot, withdraw e sub-conta.

## Execução de venda (cash-out convert)

- Ordem **spot** (venda do ativo → USDT) (ADR-001). Fill parcial: **limit IOC ~10%**.

## Sub-contas

- **Sub-conta por usuário** (ADR-002). Virtuais primeiro; Broker Program depois.

## Monitor (tempo real)

- **WebSocket privado** (PrivateOrders / PrivateDeals / PrivateAccount); polling
  só como fallback.

## Limites / taxas

- maker 0% / taker 0,05% (ADR-001). TODO: confirmar limites de saque por rede.

## Erros e normalização

- TODO: mapear erros MEXC → erros normalizados do domínio.
