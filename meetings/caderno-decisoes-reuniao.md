# Caderno de Decisões — Reunião Lunium

**Para que serve esta reunião:** responder, de uma vez, todas as perguntas que ainda
estão em aberto no projeto — para a gente **fechar o planejamento** e **começar a
programar o sistema**.

- **Data:** ____ / ____ / 2026
- **Quem participa:** Guilherme (CEO), Mateus (operação/corretoras), Igor (investidor), CTO
- **De onde vieram as perguntas:** uma revisão de tudo o que ainda está marcado como
  "a decidir" no projeto, somada à auditoria de engenharia.

> **O que vamos construir primeiro (decidido):** o **cash-out** — o cliente **entrega
> cripto e recebe em real (PIX)**. Dois caminhos: o **modo FAST** (o cliente manda
> USDT em Polygon/Solana/Tron direto pra SmartPay, que paga o PIX em ~10s) e o **modo
> normal** (outras moedas passam pela corretora pra converter, mais barato e mais
> lento). **Comprar cripto (cash-in) fica para depois.**

> Escrevi tudo em linguagem simples. Onde aparece uma palavra do "mundo cripto/
> jurídico", ela vem explicada ali mesmo, entre parênteses.

---

## Glossário de bolso (leia 1 minuto antes)

- **Cash-out** — o nosso produto: o cliente **entrega cripto** e **recebe em real
  (PIX)**. É o que vamos construir primeiro.
- **SmartPay** — a empresa que **paga o PIX em real** ao cliente. É a peça central do
  nosso produto.
- **Corretora (MEXC)** — a "casa de câmbio". Usada só no **modo normal**, pra
  **converter** a moeda do cliente em USDT antes de mandar pra SmartPay.
- **USDT** — uma cripto "dólar digital" (vale ~1 dólar). É a moeda que a SmartPay
  recebe pra pagar o PIX.
- **Modo FAST** — o cliente manda **USDT (nas redes Polygon, Solana ou Tron)** direto
  pra SmartPay → PIX em ~10s. Rápido, taxa maior.
- **Modo normal (convert)** — outras moedas → entram na corretora → viram USDT → vão
  pra SmartPay → PIX. Mais lento, taxa menor.
- **On-chain** — quando a transferência de cripto acontece de verdade na rede.
- **KYB** — a checagem que a **corretora faz da nossa empresa** (CNPJ).
- **KYC** — a checagem de identidade **do cliente** (documento, selfie).
- **VASP** — categoria de "empresa que mexe com ativos virtuais" que o **Banco Central
  regula**.
- **Compliance** — estar **dentro das regras** (Banco Central, Receita, leis
  anti-crime). É o ponto que mais pode travar o projeto se for ignorado.

---

## Como usar este caderno

Cada pergunta tem um número (P1, P2, …), o porquê dela importar e, quando ajuda,
**opções**. Há **dois tipos**:

- **DECIDIR** — depende de vocês, sócios. Sair da reunião com a resposta escrita.
- **DESIGNAR** — não dá pra resolver na mesa (abrir conta, ler contrato, falar com
  advogado/contador). Sair com **um responsável e um prazo**.

**Selos:** 🔴 trava começar a operar com dinheiro real · ⛔ irreversível · 💰 mexe
com quanto ganhamos/gastamos · ⚠️ é um risco · 🔎 precisa buscar um dado por fora.

[PAGEBREAK]

## Bloco 1 — As decisões que NÃO dão pra desfazer (resolver primeiro)

### P1 — Em qual CNPJ vamos abrir a conta na corretora (MEXC)? ⛔🔴 DECIDIR
**O que é:** a corretora verifica a empresa antes de liberar a conta (o **KYB**). Ela
**amarra uma conta por CNPJ** e **não deixa trocar depois**. Usamos a corretora no
**modo normal** (converter a moeda do cliente em USDT).

**Por que importa:** CNPJ errado = recomeçar do zero, perdendo semanas. E sem ela o
modo normal não funciona.

- **CNPJ / empresa escolhida:** _______________________________________
- **Quem vai abrir a conta:** ______________  **Até quando:** ____________

### P2 — A conta da SmartPay sai na mesma empresa (CNPJ)? ⛔ DECIDIR
**O que é:** a SmartPay é quem paga o PIX ao cliente — a peça central. A conta dela
também é uma conta empresarial nossa.

**Por que importa:** alinhar a SmartPay e a corretora **na mesma empresa** simplifica
a contabilidade, a conferência de valores e o lado jurídico.

- **Decisão:** _______________________________________________________
- **Quem cuida:** ______________  **Até quando:** ____________

---

## Bloco 2 — Regras do governo (🔴 trava começar a operar)

> Este é o ponto que mais pode inviabilizar o projeto. E **no cash-out é ainda mais
> sensível**: estamos **pagando dinheiro em real** para quem entrega cripto — é
> exatamente o caminho que as regras anti-lavagem mais vigiam. A parte técnica o
> sistema já está preparado pra encaixar; falta a **decisão de como nos posicionar**.
> É bem provável que precise de **advogado especializado**.

### P3 — Quem confere a identidade do cliente: a Lunium ou a empresa que usa a Lunium? 🔴 DECIDIR
**O que é:** o **KYC** é a verificação de quem é a pessoa (documento, selfie). Como
vamos vender nossa estrutura também pra **outras empresas** usarem, quem faz essa
checagem — nós ou o cliente empresarial?

**Por que importa:** **quem paga o dinheiro costuma ser o responsável** perante o
governo. Muda como o sistema é construído e quem assume o risco legal.

- **Decisão:** _______________________________________________________
- **Quem cuida:** ______________  **Até quando:** ____________

### P4 — Precisamos nos registrar como "empresa de ativos virtuais" (VASP)? 🔴⚠️ DESIGNAR (advogado)
**O que é:** o Banco Central regula empresas que compram/vendem cripto. Precisamos
saber **se** e **quando** registrar, e como classificar juridicamente o nosso fluxo
(receber cripto e pagar em real).

- **Quem busca a orientação (advogado/consultoria):** ___________________  **Prazo:** _____

### P5 — Regras anti-lavagem: o que precisamos cumprir? 🔴 DECIDIR + DESIGNAR
**O que é:** a lei obriga a (a) **checar se o cliente está em listas de pessoas
proibidas** (criminosos, sancionados, "politicamente expostos"), (b) **monitorar
operações suspeitas** e (c) **reportar ao COAF** (órgão do governo). Existem empresas
que fazem essas checagens pra gente.

- **Direção decidida:** ________________________________________________
- **Empresa que fará as checagens:** _____________  **Quem avalia opções:** _________

### P6 — Precisamos checar o destino antes de pagar o PIX? ⚠️ DECIDIR
**O que é:** antes de pagar, dá pra (e às vezes a lei exige) **validar a chave PIX e
quem a recebe** — confirmar que o dinheiro não vai pra alguém em lista de proibidos.

- **Decisão (vamos checar a chave PIX / o destinatário? em quais casos?):** _______________

### P7 — Obrigações com a Receita (impostos e declarações) 🔎 DESIGNAR (contador)
- **Responsável (contador):** ______________  **Prazo:** ____________

[PAGEBREAK]

## Bloco 3 — Preço e contas (quanto a gente ganha em cada operação)

### P8 — Quanto a Lunium ganha, e a diferença de taxa entre FAST e normal? 💰 DECIDIR
**O que é:** em cada operação a gente embute um ganho. E como há dois caminhos com
custos diferentes, há **duas taxas**:

- **FAST** (USDT direto pra SmartPay) — rápido, **taxa maior** (o cliente paga mais
  pela velocidade).
- **Normal** (passa pela corretora) — mais lento, **taxa menor**.

O ganho fica embutido no valor em real que o cliente recebe (transparente na cotação).

- **FAST — forma e número:** ____________________  **Normal — forma e número:** ____________________

### P9 — Antes de fixar as taxas (P8), alguém precisa somar TODOS os custos 💰⚠️ DESIGNAR
**O que é:** o modo normal tem custo de **vender na corretora** (~0,05%), de **enviar
o USDT** pela rede, e o **risco de o preço da cripto cair** entre a hora que cotamos e
a hora que vendemos. O FAST tem o custo da SmartPay.

**Por que importa:** **se a taxa for menor que esses custos, a gente perde dinheiro em
cada operação.** Alguém precisa montar a "planilha de custo por operação" dos dois modos.

- **Quem monta a planilha de custos:** ____________  **Prazo:** _________

### P10 — Qual o valor mínimo e o máximo por operação? 💰🔎 DECIDIR (depende dos dados de P19/P20)
**O que é:** o **mínimo não é uma escolha livre** — é o **maior** entre: o mínimo que
a SmartPay paga, o mínimo de venda na corretora (modo normal), e o mínimo pra
transferir aquela cripto na rede. O **máximo** é escolha de risco.

- **Mínimo (depois dos dados):** ____________  **Máximo:** ____________

### P11 — Por quanto tempo a cotação fica "travada" e quem segura o risco de preço? 💰⚠️ DECIDIR
**O que é:** quando o cliente vê "envie X de cripto, receba R$ Y", esse valor vale por
um tempo. No **modo normal**, entre o cliente enviar a cripto e a gente vender, **o
preço pode cair** — e quem segura essa diferença é a Lunium. (No FAST quase não há
esse risco, porque USDT é estável.)

**Decisão:** qual o prazo da cotação, e a regra quando o cliente deposita atrasado ou o
preço se mexe muito — **honramos o valor cotado** ou **recotamos** ao preço do momento?

- **Prazo da cotação:** ________  **Regra de preço (honra/recota):** ____________________

---

## Bloco 4 — Dinheiro e imprevistos

### P12 — A SmartPay paga o real do bolso dela, ou precisamos deixar saldo adiantado? 💰⚠️ DESIGNAR
**O que é:** quando o USDT chega na SmartPay, ela paga o PIX ao cliente. A pergunta:
**ela usa o dinheiro dela e acerta com a gente depois**, ou exige que a gente deixe um
**saldo em real adiantado** (pré-funding) com ela?

**Por que importa:** isso define **quanto capital nosso fica preso**. Se ela frenta o
real, o nosso produto é bem "leve" de caixa — grande vantagem.

- **Como funciona (descobrir com a SmartPay):** ____________  **Quem confirma:** _________

### P13 — E quando a corretora não consegue vender tudo de uma vez? ⚠️💰 DECIDIR
**O que é:** no modo normal, às vezes a corretora vende só parte da cripto naquele
preço. A diferença pequena o sistema tolera; se for grande, a operação **fica "em
análise" pra uma pessoa decidir**.

**A decisão:** regra padrão quando falta muito — **pagar proporcional ao que vendeu**,
**vender o resto** (a preço novo, risco nosso) ou **devolver a cripto**?

- **Tolerância automática (ex.: até 10%):** ________  **Regra padrão:** ________________

### P14 — Se algo der errado depois do cliente enviar a cripto, qual a regra de devolução? ⚠️ DECIDIR
**O que é:** no cash-out, se a operação não puder ser concluída, a gente **devolve a
cripto** que o cliente enviou (não um PIX). Precisamos de uma regra e de um prazo, e do
critério de **quando tentar de novo vs. devolver**.

- **Prazo de devolução:** ____________  **Critério tentar-de-novo vs. devolver:** ____________________

[PAGEBREAK]

## Bloco 5 — Corretora e SmartPay: dados a confirmar

> Quase tudo aqui é **DESIGNAR**. Sair com **responsável + prazo** pra cada item.

### P15 — Confirmar que as "contas-filhas" da corretora servem pro nosso modelo ⚠️🔎 DESIGNAR (importante)
**O que é:** pra saber **qual cliente enviou qual depósito**, a ideia é criar **uma
"sub-conta" (conta-filha) por cliente** na corretora. Precisamos confirmar que (1)
elas recebem depósito e fazem saque sozinhas e (2) o **contrato da corretora permite**
guardar/movimentar dinheiro de terceiros assim.

**Por que importa:** se o contrato não permitir, a corretora pode **congelar a conta**.
É uma suposição central — checar **antes** de programar em cima.

- **Quem verifica:** ______________  **Prazo:** ____________  **Resultado:** ________

### P16 — Aceitamos depender de uma única corretora no começo? ⚠️ DECIDIR
**O que é:** no modo normal, toda conversão passa por **uma corretora só** (a MEXC).
Ter uma segunda fica pra depois.

- **Aceitamos? (sim/não + alguma mitigação):** _________________________________

### P17 — Pegar as informações técnicas da corretora 🔎 DESIGNAR
**O que é:** dados pra ligar o sistema à corretora (chaves de acesso, como receber
depósito, como vender, como sacar o USDT, limites por rede).

- **Quem extrai:** ______________  **Prazo:** ____________

### P18 — Abrir a conta da SmartPay e pegar os dados dela 🔎 DESIGNAR
**O que é:** credenciais e dados da SmartPay (como ela nos dá o endereço de depósito,
como ela nos avisa que pagou o PIX, quais redes/moedas aceita, limites, prazo de
pagamento). É a peça central do MVP.

- **Quem abre/extrai:** ______________  **Prazo:** ____________

---

## Bloco 6 — Caixa do projeto

### P19 — De quanto capital a gente precisa, e por quanto tempo o dinheiro dura? 💰 DECIDIR
**O que é:** boa notícia — o cash-out é **leve de caixa**: o **cliente traz a cripto**
e (se a SmartPay frentar o real — ver P12) **não precisamos de um estoque grande**. O
gasto principal vira **desenvolvimento e infraestrutura**. Preciso saber quanto dos
aportes (Igor US$ 20 mil; CTO US$ 3 mil) vai pra cada coisa e por quanto tempo segura.

> **Custo fixo de infraestrutura está resolvido e baixo:** o sistema roda num servidor
> próprio simples (~US$ 5 a 10/mês) — sem o custo de US$ 100/mês que travou o btcnopix.

- **Desenvolvimento/infra:** ________  **Reserva:** ________  **Dura quantos meses:** ________

---

## Bloco 7 — Site e contas

### P20 — O site comprecripto.io está registrado, e qual o plano de "virada"? 🔎 DESIGNAR
- **Está registrado? (sim/não):** ________  **Responsável pela virada:** __________

### P21 — Quem abre cada conta e até quando 🔎 DESIGNAR
**O que é:** SmartPay (paga o PIX) e MEXC (converte, no modo normal, com saque
habilitado e contas-filhas). (Conta da Eulen só na fase de "comprar cripto", depois.)

- **SmartPay — responsável/prazo:** ______________  **MEXC — responsável/prazo:** ______________

[PAGEBREAK]

## Apêndice A — O que o CTO resolve sozinho (vocês podem ignorar)

> Decisões **técnicas internas**, que eu resolvo enquanto programo. **Não dependem de
> vocês e não travam a reunião.**

- Quais ferramentas e versões usar (banco de dados, linguagem, testes automáticos).
- Os tempos internos de espera/nova tentativa e o nº de confirmações on-chain (defino
  a partir dos dados reais das contas, quando P17/P18 chegarem).
- Como o sistema garante que nada se perde nem é pago em dobro internamente.
- **Onde o sistema roda e o IP fixo de segurança** — já resolvido: servidor próprio
  simples (~US$ 5–10/mês), sem o custo da Vercel que travou o btcnopix.

---

## Apêndice B — Checklist: "fechamos o planejamento, dá pra programar"

- [ ] **P1 e P2** decididos → contas da MEXC e da SmartPay em processo de abertura.
- [ ] **Bloco 2 (regras do governo)** com direção decidida e advogado/contador acionados.
- [ ] **P8 a P11** definidos (ou ao menos a planilha de custos de P9 com responsável) → fecha preço e risco.
- [ ] **P12 a P14** definidos → fecha como a SmartPay paga, imprevistos e devolução.
- [ ] **P15 a P18** com responsável e prazo → dados técnicos chegando.
- [ ] **P19** define o caixa.
- [ ] Cada decisão foi **registrada** no projeto, e a **ata** aponta pra elas.

**Assim que P1 (corretora) e o Bloco 2 (regras do governo) estiverem encaminhados, e
P8 a P14 decididos, eu começo a programar. As partes que dependem dos dados das contas
(P15 a P18) eu adianto com simulações até os dados reais chegarem.**
