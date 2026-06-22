# Provedor: MEXC — Exchange (spot + saque)

> Doutrina do provedor MEXC como capability `exchange` (venda spot e saque
> on-chain). Confirmado para o MVP (caminho convert).
>
> **Modelo:** conta master única + rastreamento por txId (ADR-017, supercede ADR-002).

## Papel (capability) — cash-out (ADR-016)

No caminho **convert**: recebe o depósito do cliente na conta master (identificado
por txId on-chain), vende spot → USDT (`sell_spot`, ADR-001) e saca o USDT para a
SmartPay (`withdraw_onchain`). Também fornece catálogo dinâmico (`getCatalog`).

O caminho **FAST** (USDT direto p/ SmartPay) **não toca a MEXC**.

Cash-in / `buy_spot` é fase 2.

## Credenciais e autenticação

- Conta **institucional (KYB)** — ver `runbooks/onboarding-mexc-broker.md`.
- **Auth HMAC-SHA256:**
  - Header: `X-MEXC-APIKEY: <apiKey>`
  - `totalParams` = query string (todos os params, incluindo `timestamp`)
  - `signature = HMAC-SHA256(secretKey, totalParams)` em hex minúsculo
  - `signature` é adicionado como último parâmetro na query string
  - Endpoints GET e POST: params sempre na query string (não no body)
- Credenciais via env: `MEXC_API_KEY` + `MEXC_SECRET_KEY`

## Endpoints implementados

| Método | Endpoint | Notas |
|---|---|---|
| `getCatalog` | `GET /api/v3/capital/config/getall` | Filtra redes com `depositEnable && withdrawEnable`; `withdrawIntegerMultiple` define precisão |
| `getDepositAddress` | `GET /api/v3/capital/deposit/address` | Conta master; params: `coin`, `network` |
| `getDeposit` | `GET /api/v3/capital/deposit/hisrec` | Filtra por `txId`; janela 24h; `confirmTimes = "atual/necessário"` |
| `sellSpot` | `POST /api/v3/order` | `type=IOC`, `side=SELL`, `newClientOrderId` para idempotência |
| `getOrder` | `GET /api/v3/order` | `orderId` ou `origClientOrderId` |
| `withdrawOnchain` | `POST /api/v3/capital/withdraw` | `withdrawOrderId` para idempotência; `addressTag` para redes com memo |
| `getWithdrawal` | `GET /api/v3/capital/withdraw/history` | Filtra por `id`; janela 24h |
| `getBalance` | `GET /api/v3/account` | Filtra `balances[]` por asset |

## Normalização de redes

MEXC usa nomes próprios; o domínio usa lowercase:

| Domínio | MEXC |
|---|---|
| `polygon` | `MATIC` |
| `solana` | `SOL` |
| `tron` | `TRX` |
| `btc` | `BTC` |
| `eth` | `ETH` |
| `bsc` | `BSC` |

## Execução de venda (cash-out convert)

- Ordem **spot IOC** (ADR-001): `POST /api/v3/order` com `type=IOC`, `side=SELL`
- `newClientOrderId = "{cashOutId}:sell"` — idempotência nativa MEXC
- Se a MEXC retornar `-2010 DUPLICATE_CLIENT_ORDER`, o adapter busca a ordem
  existente por `origClientOrderId` e retorna o resultado (reentrada segura)
- `cummulativeQuoteQty` = USDT recebido (quantidade preenchida do lado quote)
- `executedQty` = base vendida
- Fill < 90% → `MANUAL_REVIEW` (ADR-001, tolerância de 10%)

## Saque on-chain

- `POST /api/v3/capital/withdraw`
- `withdrawOrderId = "{cashOutId}:forward"` — idempotência
- `addressTag` obrigatório para TRX/XRP (memo/tag)
- Allowlist de endereços de saque configurada na conta MEXC (ADR-015)

## Status de saque (normalização)

| Código MEXC | Descrição | Domínio |
|---|---|---|
| 0 | email_sent | `requested` |
| 1 | cancelled | `failed` |
| 2 | awaiting_approval | `requested` |
| 3 | rejected | `failed` |
| 4 | processing | `processing` |
| 5 | failure | `failed` |
| 6 | completed | `completed` |

## Erros mapeados

| Código | Significado | Ação |
|---|---|---|
| `-2010` | `DUPLICATE_CLIENT_ORDER` | Busca ordem existente (idempotência) |
| `-2013` | `ORDER_NOT_FOUND` | Retry ou MANUAL_REVIEW |
| `30005` | `INSUFFICIENT_BALANCE` | Alerta operador |
| `700003` | `SYMBOL_HALTED` | MANUAL_REVIEW |
| `126007` | `NETWORK_SUSPENDED` | MANUAL_REVIEW |
| `126002` | `ADDRESS_NOT_ALLOWLISTED` | Verificar allowlist (ADR-015) |

## Conta master vs Broker Program (ADR-017)

MVP usa conta master única. Endereço de depósito é o mesmo para todos os usuários
do mesmo ativo/rede; identificação por txId on-chain. Quando o Broker Program for
aprovado, o `MexcAdapter` é trocado por `MexcBrokerAdapter` sem tocar no domínio.

## Taxas

- Spot: maker 0% / taker 0,05%
- Saque: depende da rede/ativo — catálogo `withdrawFee` por rede

## Monitor em tempo real

WebSocket privado (`PrivateOrders` / `PrivateDeals` / `PrivateAccount`) — a
implementar como melhoria pós-MVP. MVP usa polling.
