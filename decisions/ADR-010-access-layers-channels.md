# ADR-010 — Camadas de acesso e canais

> Como diferentes públicos acessam a operação, por quais canais, e o que cada um
> pode ver/fazer — incluindo a decisão do bot operador para o Guilherme.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-12
- **Decisores:** Guilherme, CTO

## Contexto

O mercado (Eulen, DePix, etc.) opera muito via bot de Telegram, e surgiu uma
necessidade real e **urgente, ainda não endereçada**: como o **Guilherme vê e
toca a operação** — acompanhar a integração funcionando, fazer consultas. Sem um
modelo claro, "bot", "agente" e "atendimento" se misturam, com risco de expor o
brain (estratégia/doutrina) a quem não deve.

Dois esclarecimentos orientam a decisão:
- A `lunium-api` é **headless**; qualquer bot/UI é só **mais um cliente** dela
  (mesma camada do front — ADR-005). Canal novo não mexe no núcleo.
- **Bot ≠ agente de LLM.** Um bot pode ser fluxo determinístico de menu. Ter canal
  no Telegram **não** implica puxar a camada agêntica (fase 2 — ADR-003).

## Decisão

**1. Três camadas de acesso**, com fronteira de dados explícita:

| Camada | Quem | Vê | Pode agir | Canal |
|---|---|---|---|---|
| Usuário | cliente final | só os próprios dados | só comprar | web / bot |
| Operador | Guilherme, Mateus | a operação inteira | sim (com guard-rails) | dashboard / bot operador |
| Suporte | atendimento | KB curada | não | chatbot |

**2. Fronteira inegociável do brain:** o brain só alimenta **agentes internos**
(camada operador). Canais externos (usuário, atendimento) usam uma **KB curada e
pública derivada do brain — nunca o brain cru** (evita vazar estratégia/doutrina;
mesma lógica do repo privado).

**3. Interface do operador (urgente):** começa como **bot operador determinístico
(Telegram), read-mostly** — alertas (ex.: operação em `FAILED`) e consultas
("quantas hoje?", "tem alguma travada?"). É a primeira versão concreta do
"dashboard agêntico de controle" do kickoff, **sem IA ainda**. A versão
**agêntica** (controle por linguagem natural) é **fase 2** (ADR-003).

**4. Ver vs. agir em trilhas separadas:** ver/consultar é baixo risco. Agir
(refund/retry/pause) exige **auth forte + confirmação explícita + trilha de
auditoria** — agente nunca move dinheiro sozinho (ADR-003). No início o bot é só
de leitura; ação vem depois, atrás desses controles.

## Consequências

- A `lunium-api` precisa expor **endpoints de operador/admin** e um **stream de
  eventos** (transições de estado) para o bot consumir. O bot é **cliente
  separado**, não parte do núcleo.
- Suporte e canal de usuário exigem **KB curada** à parte (futuro) — não apontar
  nenhum deles para o brain.
- O **canal de venda no Telegram** (próprio e como produto-brinde para parceiros/
  clientes Lunium) fica como **plano futuro** — ver `roadmap.md`.

## Alternativas consideradas

- **Só dashboard web para o Guilherme** — bom, mas não cobre o "me avisa quando
  travar" nativo do Telegram. O bot complementa o dashboard, não o substitui.
- **Operador já agêntico (LLM)** — descartado no MVP: complexidade/risco da fase 2
  sem necessidade; o determinístico já entrega a visibilidade.
- **Apontar bot de cliente/atendimento para o brain** — descartado: vaza estratégia.
