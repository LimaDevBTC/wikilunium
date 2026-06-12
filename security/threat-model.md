# Modelo de ameaças

> Ameaças, superfícies de ataque e mitigações do cash-in — pré-condição de merge
> (Lunium.md princípio 4).

> **STATUS: rascunho — revisar**

## Ativos a proteger
- **Chaves/credenciais** de provedor (Eulen, MEXC) e do sistema.
- **Fundos em trânsito** (fiat recebido, cripto comprada, saque on-chain).
- **Dados do usuário** (carteira de destino, valores, identidade do pedido).
- **Integridade do estado** (a máquina de estados não pode pular nem duplicar etapas).
- **Conformidade regulatória** (KYC/AML/PLD) — mover PIX→cripto p/ terceiros expõe
  a obrigações; tratar como ativo a proteger, não detalhe. Ver ADR-012 / pauta.

## Superfícies de ataque
- **API headless** exposta ao front (comprecripto-app).
- **Webhook da Eulen** (entrada externa, autenticar sempre).
- **Integrações de provedor** (respostas da MEXC/Eulen não são confiáveis por padrão).
- **Infra**: fila (BullMQ/Redis) e banco (Postgres).

## Ameaças e mitigações
- **Comprometimento de chave** → segredos em cofre, rotação periódica
  ([`../runbooks/key-rotation.md`](../runbooks/key-rotation.md)), nunca commitar segredo (`.gitignore`).
- **Webhook forjado / replay (Eulen)** → verificação de assinatura + idempotência
  por `charge_id`; webhook duplicado é no-op (ADR-007/008).
- **Dupla execução / replay de job** → idempotência por etapa na fila (ADR-007);
  cada transição numa transação ACID (ADR-004).
- **Comingling de fundos** → sub-conta por usuário (ADR-002): segregação nativa e
  depósito identificável.
- **Saque para rede errada ou suspensa** → pré-cheque `withdraw_network_enabled`
  contra o catálogo; se a rede cai após a compra, vai para `WITHDRAW_BLOCKED`
  (cripto retida, auto-resume ou refund) — não fica preso em falha sem saída.
- **Saque para endereço sancionado / travel-rule (AML)** → screening do endereço
  de saída antes de sacar; gate de compliance reservado (ADR-012). **Política
  pendente** — risco aberto até a reunião; é o ponto cego nº 1.
- **Falha após o pagamento (dinheiro retido)** → estados de retenção explícitos
  (`AWAITING_LIQUIDITY`/`WITHDRAW_BLOCKED`/`MANUAL_REVIEW`) + outbox (ADR-013) +
  [`../runbooks/reconciliation.md`](../runbooks/reconciliation.md); dinheiro nunca "some" em silêncio. `FAILED` só por write-off revisado.
- **Oversell de estoque (vender o mesmo saldo 2x)** → reserva de liquidez /
  available-to-promise (ADR-014), não saldo bruto.
- **Movimento de dinheiro por engano** → execução 100% determinística; nenhum
  agente move fundos (ADR-003).

## Continuidade (lição do ambiente anterior)
No projeto anterior, o conhecimento operacional vivia numa só pessoa; quando deu
problema, tudo caiu de uma vez. O modelo do Lunium é o oposto por desenho: **esta
wiki é a fonte de verdade**, então, mesmo sem o autor original, Guilherme, Mateus
e Igor continuam o projeto sem depender de ninguém. "Tudo que está só na cabeça de
alguém é um bug" (princípio 5) — é um diferencial deliberado deste trabalho.
