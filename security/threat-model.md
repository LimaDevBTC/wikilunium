# Modelo de ameaças

> Ameaças, superfícies de ataque e mitigações do **cash-out** (ADR-016) —
> pré-condição de merge (Lunium.md princípio 4).

> **STATUS: revisado em 2026-06-23**

## Ativos a proteger
- **Chaves/credenciais** de provedor (SmartPay, MEXC) e do sistema.
- **Fundos em trânsito** (cripto recebida do cliente, USDT da venda, payout em BRL).
- **Dados do usuário** (chave PIX, valores, identidade do pedido).
- **Integridade do estado** (a máquina de estados não pode pular nem duplicar etapas).
- **Conformidade regulatória** (KYC/AML/PLD) — mover PIX→cripto p/ terceiros expõe
  a obrigações; tratar como ativo a proteger, não detalhe. Ver ADR-012 / pauta.

## Superfícies de ataque
- **API headless** exposta ao front (vendacripto-app) — protegida por `CLIENT_API_TOKEN` (Bearer) + rate limiting (`@nestjs/throttler`).
- **Webhook da SmartPay** (payout — entrada externa, autenticar sempre por HMAC).
- **Endpoints de admin** (`/admin/*`) — protegidos por `ADMIN_TOKEN` (Bearer), sem rate limiting (acesso interno).
- **Depósito on-chain do cliente** (valor/rede/origem não confiáveis por padrão).
- **Integrações de provedor** (respostas da MEXC/SmartPay não são confiáveis por padrão).
- **Infra**: fila (BullMQ/Redis) e banco (Postgres) — acesso só pelo VPS (ADR-015); sem exposição pública.

## Ameaças e mitigações
- **Comprometimento de chave** → segredos em cofre, rotação periódica
  ([`../runbooks/key-rotation.md`](../runbooks/key-rotation.md)), nunca commitar segredo (`.gitignore`);
  **chaves de privilégio mínimo** (ler/negociar/sacar separadas), **allowlist de
  endereço de saque** no provedor e **IP de egress único** na allowlist (ADR-015) —
  mesmo com a chave vazada, o saque só vai para endereço pré-aprovado.
- **Webhook forjado / replay (SmartPay)** → verificação de assinatura + idempotência
  por `payout_order_id`/`cash_out_id`; webhook duplicado é no-op (ADR-007/008).
- **Depósito falso/divergente do cliente** (valor ou rede errada, ou "confirmação"
  forjada) → só agir sobre depósito **confirmado** pela fonte confiável (MEXC/chain);
  guard `deposit_matches_quote`; divergência → `MANUAL_REVIEW` (cripto irreversível).
- **Dupla execução / replay de job** → idempotência por etapa na fila (ADR-007);
  cada transição numa transação ACID (ADR-004).
- **Comingling de fundos / depósito de cliente A creditado em B** → conta master única (ADR-017); depósito identificado por `txId` on-chain + `ReconciliationService` detecta divergências automaticamente; trilha ACID de todos os eventos em `CashOutEvent`.
- **Saque (USDT→SmartPay) para rede suspensa** → pré-cheque de rede no catálogo;
  falha cai em `MANUAL_REVIEW` (cripto/USDT retido, retoma ou devolve) — sem beco.
- **Payout para chave PIX errada / sancionado / travel-rule (AML)** → screening
  **antes do payout** (ADR-012) + validação da chave PIX. **Política pendente** —
  risco aberto até a reunião; é o ponto cego nº 1 (mais grave no cash-out).
- **Recebido, não pago (cripto do cliente retida)** → `MANUAL_REVIEW` + `REFUNDING_
  CRYPTO` + outbox (ADR-013) + [`../runbooks/reconciliation.md`](../runbooks/reconciliation.md);
  nunca "some" em silêncio. `FAILED` só por write-off revisado.
- **Movimento de dinheiro por engano** → execução 100% determinística; nenhum agente move fundos (ADR-003). Ações de operador em `MANUAL_REVIEW` exigem autenticação explícita (`ADMIN_TOKEN`).
- **Abuso / DDoS na API pública** → rate limiting por IP (`@nestjs/throttler`): 5 req/min em `POST /cash-outs`, 10 em `/accept`, 60 em `GET /:id`. Webhook e admin isentos (fontes confiáveis ou autenticadas).
- **Token de cliente vazado** → rotacionar `CLIENT_API_TOKEN` (ver [`runbooks/key-rotation.md`](../runbooks/key-rotation.md)); sem necessidade de IP fixo no cliente — token basta.

## Continuidade (lição do ambiente anterior)
No projeto anterior, o conhecimento operacional vivia numa só pessoa; quando deu
problema, tudo caiu de uma vez. O modelo do Lunium é o oposto por desenho: **esta
wiki é a fonte de verdade**, então, mesmo sem o autor original, Guilherme, Mateus
e Igor continuam o projeto sem depender de ninguém. "Tudo que está só na cabeça de
alguém é um bug" (princípio 5) — é um diferencial deliberado deste trabalho.
