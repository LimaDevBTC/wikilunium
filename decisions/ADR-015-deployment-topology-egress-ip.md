# ADR-015 — Topologia de implantação e IP de egress (provedores com allowlist)

> Onde cada parte roda, e como atender provedores que exigem IP fixo na allowlist
> **sem** pagar IP estático de serverless nem exigir IP fixo dos clientes.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-21
- **Decisores:** Guilherme, CTO
- **Origem:** lição do **btcnopix** — a GetMoons exigia **IP fixo** para liberar a
  API; hospedado na **Vercel (serverless)**, IP estático custa ~US$100/mês, e
  ~US$120/mês de custo inviabilizou o produto.

## Contexto

Provedores de cripto/pagamento frequentemente protegem a API por **allowlist de
IP** (a MEXC, p.ex., permite vincular a chave a um IP **e** restringir endereços
de saque). Plataformas **serverless** (Vercel) não dão um **IP de saída estável**
barato — vira um add-on caro. O btcnopix tentou rodar o componente que fala com o
provedor em serverless e bateu nesse custo.

Fato que dissolve o problema na Lunium: a `lunium-api` (ADR-009) é **NestJS +
workers BullMQ + Redis + monitor WebSocket** — um serviço **long-running**, que
**não roda em função serverless** de qualquer forma. Ele precisa de um host
persistente — e host persistente **já vem com IP estático** de graça/barato.

## Decisão

**Separar o keyholder do front, e centralizar o egress:**

- **`lunium-api` (keyholder)** roda em **VPS/contêiner persistente** com **IPv4
  estático nativo** (~US$5–10/mês). É onde vivem as chaves de provedor, os
  workers, a fila, as migrations e o monitor. Esse **único IP** é o que se
  cadastra na allowlist de cada provedor. **Serverless está descartado** — três
  cargas que não cabem numa função efêmera: (1) **listener de confirmações
  on-chain** (processo vivo, não morre após a request); (2) **crons baratos**
  (sincronizar ordens MEXC, varrer pendências); (3) **WebSocket longo com a MEXC**
  (conexão persistente). Reforça ADR-009.
- **Front (`vendacripto-app`) na Vercel** hospeda **só a UI**. **Nunca** detém
  chave de provedor nem chama provedor direto — só chama a `lunium-api` por
  **HTTPS com token**. Logo **o front não precisa de IP fixo**.
- **Egress por um IP só:** todo tráfego de saída para provedores sai pelo IP da
  `lunium-api`. **Um IP serve todos os clientes B2B — nenhum cliente precisa de
  IP fixo** (princípio 1 / ADR-005: cliente nunca toca o provedor).
- **Plano B (evitar):** se algum componente serverless precisar falar com
  provedor, rotear a saída por um **proxy de IP fixo** (WireGuard/Tailscale exit
  num VPS, IPv4 dedicado Fly ~US$2, ou QuotaGuard/Fixie). Desnecessário com o
  keyholder no VPS.

**Mesma segurança (ou maior) sem depender só do IP** — camadas combinadas:
chaves de **privilégio mínimo** (ler/negociar/sacar separadas; a de saque
isolada no worker), **allowlist de endereço de saque** no provedor (mais forte
que IP no pior caso), **assinatura HMAC**, segredos em cofre (`security/`),
verificação de assinatura de webhook, rotação (`runbooks/key-rotation.md`). O IP
allowlist vira **uma camada barata**, não o pilar único.

## Consequências

- Custo de infra do keyholder ~**US$5–10/mês** (VPS com IP), não US$100 — resolve
  a inviabilidade do btcnopix. Entra no orçamento (`caderno`/runway).
- Workers/fila/migrations/monitor vivem no host persistente; o front é deploy
  separado (repos já são separados — ADR-005). O **front (Next.js) pode ficar na
  Vercel free** (sem chave, sem IP) ou no mesmo VPS — escolha menor.
- Front↔API por HTTPS + token (CORS/escopo). A allowlist de IP do provedor aponta
  para o IP do host (rotação de IP = atualizar a allowlist — runbook).
- **Host recomendado:** Hetzner Cloud CX22 (~€4–5/mês, IPv4 incluído) pelo
  custo/benefício; DigitalOcean/Vultr como alternativa. No MVP, **um VPS roda tudo**
  (API + worker + Postgres + Redis via `docker-compose`); separar conforme o volume.
- **Responsabilidades que a Vercel abstraía e agora são nossas:** **backup
  automático do Postgres** (sistema de dinheiro — inegociável), firewall (só 443 +
  SSH), deploy reproduzível (CI/`docker-compose`), e documentar o IPv4 na allowlist
  de cada provedor. Postgres gerenciado (backup/HA) é caminho de evolução pós-MVP.
- &lt;TODO: confirmar a região do VPS — perto do usuário/PIX (BR) vs. perto dos
  endpoints da MEXC se a latência de execução pesar; decisão de otimização, não de MVP.&gt;

## Alternativas consideradas

- **IP estático na Vercel (~US$100/mês)** — descartado por custo e porque não
  resolve o workload long-running (a `lunium-api` não cabe em serverless).
- **Serverless sem IP fixo + provedor sem allowlist** — descartado: perde uma
  camada de segurança e nem todo provedor aceita (a GetMoons exigia IP).
- **Proxy de IP fixo como SaaS (QuotaGuard/Fixie)** — viável como plano B; mais
  simples hospedar o keyholder num VPS que já tem IP.
