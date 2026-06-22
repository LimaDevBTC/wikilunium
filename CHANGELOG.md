# Changelog

Todas as mudanças relevantes deste brain são registradas aqui.
Formato baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/).

## [Não lançado]

### Pivot do MVP para cash-out (2026-06-21)

**Mudança de escopo (ADR-016)**
- O MVP passa de **cash-in** (comprar cripto) para **cash-out** (vender cripto →
  PIX via SmartPay): caminho **FAST** (USDT em Polygon/Solana/Tron direto p/ a
  SmartPay, ~10s) e **convert** (demais via MEXC: venda spot → USDT → SmartPay).
  É capital-light (cliente traz o ativo; SmartPay frente o BRL). **Cash-in vira fase 2.**
- `domain/state-machine.yaml` **reescrita** para o cash-out (QUOTE → AWAITING_DEPOSIT
  → [FAST: payout] | [convert: DEPOSIT → SELLING → SOLD → FORWARDING] → PAYING_OUT →
  COMPLETED; exceções `MANUAL_REVIEW`, `REFUNDING_CRYPTO/REFUNDED`).
- `providers/_capability-contract.yaml`: port `off_ramp` (SmartPay: create_payout_order,
  retry_payout, verify_webhook) + `exchange` (get_deposit_address, get_deposit,
  **sell_spot**, withdraw p/ SmartPay); `on_ramp` (Eulen) → fase 2.

**Providers**
- SmartPay → **provedor confirmado do MVP**; Eulen (PIX-in) → **fase 2**; MEXC muda
  de papel (recebe depósito + **vende** spot + saca USDT p/ a SmartPay).
- ADR-011 (estoque dois-mundos) e ADR-014 (reserva anti-oversell) → **cash-in/fase 2**
  (cash-out é capital-light, sem oversell).

**Alinhamento**
- Atualizados Lunium.md (§0/§3/§4/§5/§2/§6), roadmap (faixas/fases), money-rules
  (venda, taxas FAST/convert, truncar a favor do operador), glossary, security
  (threat-model/controls — webhook SmartPay, screening antes do payout, depósito
  divergente) e runbooks (reconciliação e incidente no fluxo de venda).

### Topologia de implantação + off-ramp FAST (2026-06-21)

**Adicionado**
- ADR-015 (topologia de implantação + IP de egress): keyholder (`lunium-api`) em
  host persistente com **um IP estático** na allowlist dos provedores (~US$5/mês,
  não US$100 da Vercel — lição do btcnopix/GetMoons); front sem chave nem IP;
  nenhum cliente precisa de IP fixo. Segurança por camadas (privilégio mínimo +
  allowlist de saque + HMAC + cofre), não só IP.
- `providers/smartpay.md`: doutrina do off-ramp com **modo FAST** (USDT em
  Polygon/Solana/Tron direto p/ a SmartPay, PIX em ~10s) e **modo convert** (demais
  moedas via MEXC). Escopo (MVP vs fase 2) a confirmar — pauta.
- `security/` (controls + threat-model): chaves de privilégio mínimo, allowlist de
  endereço de saque, IP de egress único; `glossary.md`: off-ramp/modo FAST,
  IP de egress, keyholder.

### Auditoria de engenharia (2026-06-12) — correções e novas decisões

**Adicionado**
- ADR-012 (compliance/KYC-AML — **Proposto**): reserva o gate de screening na
  máquina (`screening_passed`); política pendente de reunião, bloqueia o go-live.
- ADR-013 (outbox transacional): consistência fila↔banco; elimina ordem travada /
  job fantasma do dual-write (ADR-004 + ADR-007).
- ADR-014 (reserva de liquidez / available-to-promise): pré-cheque anti-oversell.
- `state-machine.yaml`: estados de exceção `WITHDRAW_BLOCKED`, `MANUAL_REVIEW`,
  `REFUNDING`, `REFUNDED`; caminho de under-fill (`FILL_UNDER_TOLERANCE`); chaves
  de idempotência definidas por etapa; seam de screening e re-cheque de rede.
- `money-rules.md`: regra de tipo monetário (decimal/bigint, nunca float JS) e
  tratamento explícito de under-fill.
- `_capability-contract.yaml`: método `on_ramp.refund_pix`; `cross_cutting`
  preenchido (idempotência, outbox, retry, normalização de erro).
- `roadmap.md`: reaper de estados travados na Fase 3; **gate de go-live canário**
  com limites duros; dependência crítica KYB↔decisão da entidade explicitada.

**Corrigido**
- Backfill de ADR-001, ADR-002 e ADR-003 (Contexto/Consequências/Alternativas
  estavam vazios — feria o princípio 5).
- Drift de nome `liquidity_failed` → `AWAITING_LIQUIDITY` (liquidity-model.md).
- Refund agora é representável na máquina (antes: `FAILED` terminal sem saída).
- `security/` (threat-model + controls): AML/screening, endereço sancionado,
  oversell, outbox; `glossary.md` e runbooks alinhados aos novos estados.

**Pendente de reunião** (movido para `meetings/pauta-decisoes-pendentes.md`):
política de compliance, parâmetros de reserva/oversell, tolerância de under-fill,
verificação MEXC (sub-contas/ToS) + risco de concentração, mecanismo de refund
(Eulen), TTL do quote, custo carregado das duas pontas p/ a taxa.
