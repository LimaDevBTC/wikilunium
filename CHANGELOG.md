# Changelog

Todas as mudanças relevantes deste brain são registradas aqui.
Formato baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/).

## [Não lançado]

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
