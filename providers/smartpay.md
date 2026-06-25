# Provedor: SmartPay — Off-ramp PIX (pagamento em BRL)

> Doutrina do provedor SmartPay como capability `off_ramp`: recebe **USDT** e
> **paga o cliente em BRL via PIX**. É a saída do cash-out.

> **STATUS: adapter implementado (18b1f4b) — aguarda credenciais reais (Faixa B)**
>
> Swagger: `https://wantspay.truther.sv/api/v1-json`  
> Docs: `https://wantspay.truther.sv/docs`  
> Adapter: `src/adapters/smartpay/smartpay.adapter.ts`

---

## Papel (capability)

- Implementa: `off_ramp` — entra cripto (USDT), sai **BRL no PIX** do cliente.

---

## Dois caminhos (cash-out)

- **Modo FAST** — o cliente envia **USDT em Polygon, Solana ou Tron** → vai
  **direto para a SmartPay** → a SmartPay paga o PIX em **~10s, tudo on-chain**.
  Sem conversão, sem MEXC. Vale **somente** para esses tokens/redes.
  Taxa maior (premium pela velocidade).

- **Modo CONVERT** — qualquer outra moeda → **entra na MEXC** → convertida em USDT →
  USDT sacado (Polygon/Solana/Tron) **para a SmartPay** → SmartPay paga o PIX.
  Mais lento, taxa menor.

Escolha determinística: `asset == USDT and network in {polygon, solana, tron}` → FAST; senão → CONVERT.

---

## Autenticação (adapter)

**Tipo:** JWT Bearer  
**Endpoint:** `POST /v1/auth/login`  
**Body:** `{ "username": "<CNPJ ou email>", "password": "..." }`  
**Response:** `{ "access_token": "...", "expires_in": 3600 }`

O adapter cacheia o token e o renova automaticamente 5 min antes de expirar. Configurar via env:
```
SMARTPAY_USERNAME=<credencial>
SMARTPAY_PASSWORD=<senha>
```

---

## Fluxo off-ramp (como o adapter usa a API)

### 1. Endereços de depósito

SmartPay mantém **endereços fixos por rede** (polygon, solana, tron) para receber USDT. 
Configurar como env vars (obtidos na ativação da conta):

```
SMARTPAY_DEPOSIT_POLYGON=0x...
SMARTPAY_DEPOSIT_SOLANA=...
SMARTPAY_DEPOSIT_TRON=T...
```

O cliente (FAST) ou a MEXC (CONVERT) envia USDT para este endereço com um **memo codificado**.

### 2. Registro do source address (setup único)

Antes de usar, registrar o endereço MEXC master como `source_address` na SmartPay para que ela aceite depósitos:

```
POST /v1/application/create-source-address
Authorization: Bearer <jwt>
Body: { "network": "polygon", "address": "<endereço MEXC master>" }
```

### 3. Encode memo (por operação)

Para cada cash-out, o adapter chama `POST /v1/application/encode-memo` para codificar o memo on-chain com a instrução de pagamento:

```
POST /v1/application/encode-memo
Body: { "memo": "<chave_pix>|refund:<SMARTPAY_REFUND_ADDRESS>" }
```

O resultado (memo encodado) é passado como `tag`/`memo` na transferência on-chain de USDT.  
Configurar endereço de refund (EVM) via:
```
SMARTPAY_REFUND_ADDRESS=0x...
```

**Formato memo (MemoDataEVM):** `"<chave_pix>|refund:<evm_address>"`

### 4. Status da operação

```
GET /v1/br/payments/off-ramp/status?txid=<blockchain_txid>
Authorization: Bearer <jwt>
```

Status possíveis: `PROCESSING`, `SUCCESS`, `FAILED`, `REFUNDED`.

---

## Webhook (PIX_OFFRAMP)

SmartPay chama nosso endpoint quando o status muda. Configurar via:

```
POST /v1/application/webhook-create
Body:
{
  "callbackUrl": "https://api.vendacripto.io/webhooks/smartpay",
  "apiKey": "<SMARTPAY_WEBHOOK_API_KEY>",
  "type": "PIX_OFFRAMP",
  "active": true
}
```

**Autenticação do webhook:** o campo `apiKey` é enviado no header `x-signature`. O adapter verifica este valor. Configurar via:
```
SMARTPAY_WEBHOOK_API_KEY=<chave-gerada>
```

**TODO:** Verificar formato exato do payload do webhook quando credenciais forem configuradas. O adapter assume:
```json
{
  "txid": "<blockchain_hash>",
  "status": "SUCCESS|FAILED|PROCESSING|REFUNDED",
  "reference": "<cashOutId>",  // se SmartPay echoar nosso memo
  "e2e": "<PIX_e2e_id>"        // em SUCCESS
}
```

Endpoint Lunium que recebe: `POST /webhooks/smartpay` (autenticado por `x-signature`, isento de throttle e ClientApiToken).

---

## Env vars (todas)

| Var | Obrigatória em prod | Descrição |
|---|---|---|
| `SMARTPAY_BASE_URL` | Sim | Base URL da API (default: `https://wantspay.truther.sv`) |
| `SMARTPAY_USERNAME` | Sim | Credencial de login (CNPJ ou email) |
| `SMARTPAY_PASSWORD` | Sim | Senha de login |
| `SMARTPAY_WEBHOOK_API_KEY` | Sim | apiKey para verificar autenticidade dos webhooks |
| `SMARTPAY_REFUND_ADDRESS` | Sim | Endereço EVM para devolução em caso de falha no PIX |
| `SMARTPAY_DEPOSIT_POLYGON` | Sim (se usar polygon) | Endereço receptor SmartPay na rede Polygon |
| `SMARTPAY_DEPOSIT_SOLANA` | Sim (se usar solana) | Endereço receptor SmartPay na rede Solana |
| `SMARTPAY_DEPOSIT_TRON` | Sim (se usar tron) | Endereço receptor SmartPay na rede Tron |

---

## Erros normalizados

| Situação | Comportamento do adapter |
|---|---|
| Login falhou (HTTP !2xx) | Lança `Error("SmartPay autenticação falhou: HTTP <status>")` |
| encode-memo falhou | Lança `Error("SmartPay encode-memo falhou: HTTP <status>")` |
| Rede não configurada | Lança `Error("SmartPay: endereço de depósito não configurado para rede <rede>")` |
| getPayoutStatus HTTP !2xx | Lança `Error("SmartPay getPayoutStatus falhou: HTTP <status>")` |
| Webhook: apiKey inválida | Lança `Error("SmartPay webhook: apiKey inválida")` |
| retryPayout | No-op — retorna `{ status: 'paying' }` (SmartPay auto-processa; retry via nova operação) |

---

## Perguntas ainda em aberto (Faixa B)

- **Formato exato do payload de webhook** — o campo `reference` ecoa o nosso cashOutId/memo?
- **Limites da conta** — volume máximo por operação, por dia, por rede.
- **Prazo de liquidação** — quanto tempo entre USDT recebido e PIX confirmado.
- **Setup source-address** — precisa de registro prévio para cada endereço MEXC que vai sacar USDT?
- **Pré-funding de BRL** — SmartPay paga o PIX com seu próprio saldo antes de receber USDT, ou só após confirmação on-chain?
- **Contas de sandbox** — existe ambiente de teste antes do go-live?
