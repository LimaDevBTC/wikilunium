# Controles de segurança

> Controles concretos (acesso, segregação, auditoria, continuidade) que
> implementam o modelo de ameaças.

> **STATUS: rascunho — revisar**

## Controle de acesso e segredos
- Segredos em cofre, **fora do código**; `.gitignore` bloqueia `.env`/chaves.
- Rotação de chaves — [`../runbooks/key-rotation.md`](../runbooks/key-rotation.md). TODO: cadência.
- **Chaves de privilégio mínimo:** chaves de provedor separadas por função
  (ler / negociar / **sacar**); a chave de saque ("joia da coroa") só no worker.
- **Allowlist de endereço de saque** no provedor (a MEXC suporta): mesmo com a
  chave comprometida, o saque só vai para endereços pré-aprovados — mais forte que
  IP no pior caso.

## Egress e IP allowlist de provedor (ADR-015)
- O keyholder (`lunium-api`) roda em **host persistente com IP estático**; esse
  **único IP** entra na allowlist de cada provedor (a Vercel/serverless não dá IP
  fixo barato — lição do btcnopix). O **front nunca chama o provedor** e **nenhum
  cliente precisa de IP fixo**.
- IP allowlist é **uma camada** entre várias (chaves de privilégio mínimo +
  allowlist de saque + HMAC + cofre), não o pilar único.

## Segregação de fundos
- Sub-conta por usuário (ADR-002); reconciliação por sub-conta.

## Idempotência e atomicidade
- Chave de idempotência por etapa (ADR-007); cada transição numa transação ACID
  (ADR-004).

## Idempotência e atomicidade (cont.)
- Próximo job gravado na mesma transação da transição (**outbox** — ADR-013):
  sem ordem travada nem job fantasma.
- Pré-cheque sobre **disponível-para-prometer** (reserva — ADR-014), não saldo bruto.

## Compliance / screening
- **Gate de screening** reservado **antes do payout** em BRL (`screening_passed` —
  seam). Crítico no cash-out (cripto→fiat é vetor de lavagem). Política (KYC/AML/
  sanções, quem executa) **pendente** — ADR-012 / pauta. Quando fechar, vira
  controle ativo (não no-op).

## Auditoria e logs
- Trilha de auditoria **por transição de estado** e **por movimento de dinheiro**
  (quem/quando/quanto/resultado), apoiada na máquina de estados + outbox.
  <TODO: retenção e formato.>

## Confirmação explícita de movimento de dinheiro
- Execução determinística; agente nunca move fundos (ADR-003). Ações do operador
  que movem dinheiro (refund/retry/write-off em `MANUAL_REVIEW`) exigem auth forte
  + confirmação + auditoria (ADR-010).

## Continuidade
- Doutrina, decisões e runbooks vivem neste brain (princípio 5). Onboarding de um
  novo responsável é via `Lunium.md` + runbooks — sem dependência de uma pessoa.
