# Controles de segurança

> Controles concretos (acesso, segregação, auditoria, continuidade) que
> implementam o modelo de ameaças.

> **STATUS: rascunho — revisar**

## Controle de acesso e segredos
- Segredos em cofre, **fora do código**; `.gitignore` bloqueia `.env`/chaves.
- Rotação de chaves — [`../runbooks/key-rotation.md`](../runbooks/key-rotation.md). TODO: cadência.

## Segregação de fundos
- Sub-conta por usuário (ADR-002); reconciliação por sub-conta.

## Idempotência e atomicidade
- Chave de idempotência por etapa (ADR-007); cada transição numa transação ACID
  (ADR-004).

## Auditoria e logs
- TODO: trilha de auditoria de cada transição de estado e de cada movimento de
  dinheiro (quem/quando/quanto/resultado).

## Confirmação explícita de movimento de dinheiro
- Execução determinística; agente nunca move fundos (ADR-003).

## Continuidade
- Doutrina, decisões e runbooks vivem neste brain (princípio 5). Onboarding de um
  novo responsável é via `Lunium.md` + runbooks — sem dependência de uma pessoa.
