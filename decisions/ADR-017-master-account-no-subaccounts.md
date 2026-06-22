# ADR-017 — Conta master única (sem sub-contas MEXC) para o caminho CONVERT

**Status:** Aceito  
**Data:** 2026-06-22  
**Supercede parcialmente:** ADR-002 (sub-conta por usuário) — revogado para o MVP; mantém a intenção de segregação como objetivo de fase posterior.

---

## Contexto

ADR-002 especificou sub-conta por usuário na MEXC para segregação de fundos e
depósito identificável. Após análise da API MEXC (junho/2026):

1. **Limite duro de 30 sub-contas** por conta standard — inaceitável para escalar
   além do MVP; teto de usuários simultâneos no caminho CONVERT.
2. **API key própria por sub-conta**: para obter endereço de depósito, colocar
   ordens e sacar em nome de uma sub-conta, é necessário usar as credenciais
   _daquela_ sub-conta — a key master não basta. Isso exige armazenar N pares
   chave/secret no banco (superfície de ataque proporcional ao número de usuários).
3. **MEXC Broker Program** (sem limite de sub-contas, API key master suficiente):
   requer aprovação com volume demonstrado — indisponível no lançamento.

## Decisão

**MVP: conta master única** com identificação de depósito por `txId` on-chain.

### Modelo operacional (caminho CONVERT)

```
Cliente → endereço master (ativo/rede) → txId on-chain
    ↓
Lunium monitora hisrec da MEXC → casa por txId + asset + amount ± tolerância
    ↓
sell_spot (master) → withdraw USDT → SmartPay → PIX
```

- Todos os usuários do caminho CONVERT depositam no **mesmo endereço** da conta
  master para cada ativo/rede.
- Identificação: `txId` (único na blockchain por definição) — sem ambiguidade
  mesmo com valores idênticos simultâneos.
- Matching: `txId` + `asset` + `network` + `amount ± 1%` + cash-out em
  `AWAITING_DEPOSIT` dentro do TTL.
- **`subaccountId` removido** do agregado `CashOut` e do `ExchangePort` — sem
  efeito no domínio ou na máquina de estados.

## Consequências

**Positivo**
- Sem limite de usuários simultâneos.
- Adapter MEXC com **um único par de credenciais** (master) — superfície de
  segurança mínima; cofre guarda duas chaves, não N.
- Implementação mais simples, menos estados para rastrear.

**Negativo e mitigações**

| Risco | Mitigação |
|-------|-----------|
| Fundos de múltiplos usuários coexistem no master | Janela curta (min a horas); venda e saque imediatos após confirmação; reconciliação (Fase 4) vincula cada saldo a um cash-out por `txId`. |
| Mesmo endereço de depósito para todos | Endereço exibido uma vez por operação; TTL de cotação curto (15 min); matching por `txId`, não por endereço. |
| Usuário reutiliza endereço de cash-out anterior | Cash-out expirado não aceita novo depósito; depósito "órfão" → MANUAL_REVIEW via reaper. |

## Impactos no código

- `ExchangePort`: `subaccountId` removido de todos os métodos; `subaccountCreate`
  e `subaccountBalance` substituídos por `getBalance(asset)`.
- `CashOut` (domínio + schema Prisma): campo `subaccountId` removido.
- `PrismaPersistenceAdapter`, `CashOutOrchestrator`, `FakeMexc`: atualizados.
- `state-machine.yaml` (brain): efeito `exchange.getDepositAddress` mantém o nome;
  a semântica muda de "endereço da sub-conta" para "endereço master para ativo/rede".

## Caminho de migração para Broker Program

Quando o volume justificar, aplicar ao MEXC Broker Program. No Broker Program:
- Sub-contas reais sem limite prático.
- API key master suficiente para operar em nome das sub-contas (sem chaves adicionais).
- Migração: trocar o `MexcAdapter` para usar sub-conta por usuário; o domínio
  (`ExchangePort`, state-machine, `CashOut`) **não muda**.

O `ExchangePort` será estendido com `subaccountCreate` / `subaccountBalance`
nessa fase, sem quebrar o modelo atual.
