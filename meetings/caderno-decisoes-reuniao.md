# Caderno de Decisões — Reunião Lunium

**Para que serve esta reunião:** responder, de uma vez, todas as perguntas que ainda
estão em aberto no projeto — para a gente **fechar o planejamento** e **começar a
programar o sistema**.

- **Data:** ____ / ____ / 2026
- **Quem participa:** Guilherme (CEO), Mateus (operação/corretoras), Igor (investidor), CTO
- **De onde vieram as perguntas:** uma revisão de tudo o que ainda está marcado como
  "a decidir" no projeto, somada à auditoria de engenharia feita em 12/06/2026.

> Escrevi tudo em linguagem simples, sem termo técnico solto. Onde aparece uma
> palavra do "mundo cripto/jurídico", ela vem explicada ali mesmo, entre parênteses.

---

## Glossário de bolso (leia 1 minuto antes)

- **Cash-in** — o caminho do dinheiro no nosso produto: o cliente **paga em real (via
  PIX)** e **recebe cripto** na carteira dele. É isso que vamos construir primeiro.
- **Corretora (MEXC)** — a "casa de câmbio" onde a gente compra a cripto. No nosso
  caso, a MEXC.
- **Eulen** — a empresa que recebe o PIX do cliente pra gente. Quando o cliente paga,
  ela nos avisa e credita o valor (na forma de um "real digital", o **DePix**).
- **Estoque** — a cripto que a gente deixa **comprada e parada** na corretora, pronta
  pra entregar na hora que um cliente compra. É como o estoque de uma loja.
- **Carteira / on-chain** — o "endereço" de cripto do cliente, pra onde a gente envia
  a moeda. "On-chain" é só o nome de quando a transferência acontece de verdade na rede.
- **KYB** — a checagem que a **corretora faz da nossa empresa** (CNPJ) antes de liberar
  a conta. (Know Your Business = "conheça a empresa".)
- **KYC** — a checagem de identidade **do cliente final** (documento, selfie). (Know
  Your Customer = "conheça o cliente".)
- **VASP** — a categoria de "empresa que mexe com ativos virtuais" que o **Banco
  Central regula**. Se formos enquadrados, há regras a cumprir.
- **Compliance** — estar **dentro das regras** (Banco Central, Receita, leis
  anti-crime). É o ponto que mais pode travar o projeto se for ignorado.

---

## Como usar este caderno

Cada pergunta tem um número (P1, P2, …), o porquê dela importar e, quando ajuda,
**opções**. Há **dois tipos**:

- **DECIDIR** — depende de vocês, sócios. A ideia é **sair da reunião com a resposta
  escrita** no campo.
- **DESIGNAR** — não dá pra resolver na mesa (precisa abrir uma conta, ler um contrato,
  falar com advogado/contador). Aqui a meta é sair com **um responsável e um prazo**.

**Selos ao lado de cada pergunta:**
🔴 trava o início da operação com dinheiro real · ⛔ é irreversível (não dá pra
desfazer) · 💰 mexe com o quanto ganhamos/gastamos · ⚠️ é um risco · 🔎 precisa
buscar um dado/confirmação por fora.

[PAGEBREAK]

## Bloco 1 — As decisões que NÃO dão pra desfazer (resolver primeiro)

> São as mais demoradas e que **não dependem só da gente** (a corretora pode levar
> semanas pra aprovar e até recusar). Tudo que envolve dinheiro real depende daqui —
> por isso vêm primeiro.

### P1 — Em qual CNPJ vamos abrir a conta na corretora (MEXC)? ⛔🔴 DECIDIR
**O que é:** a corretora exige verificar a empresa antes de liberar a conta — essa
checagem é o **KYB**. Ela **amarra uma conta por CNPJ** e **não deixa trocar depois**
(nem de pessoa física pra empresa).

**Por que importa:** se escolhermos o CNPJ errado, não dá pra corrigir — teríamos que
recomeçar a conta do zero, perdendo semanas. E **sem essa conta a gente não compra nem
entrega cripto** — ou seja, trava todo o resto.

- **CNPJ / empresa escolhida:** _______________________________________
- **Quem vai abrir a conta:** ______________  **Até quando:** ____________

### P2 — A mesma empresa (CNPJ) vai abrir também a conta da Eulen? ⛔ DECIDIR
**O que é:** a Eulen é quem recebe o PIX do cliente. Ela também tem uma conta
empresarial nossa.

**Por que importa:** ter a entrada do dinheiro (Eulen) e a saída (corretora) **na mesma
empresa** simplifica muito a contabilidade, a conferência de valores e o lado jurídico.

- **Decisão:** _______________________________________________________
- **Quem cuida:** ______________  **Até quando:** ____________

---

## Bloco 2 — Regras do governo (🔴 trava começar a operar)

> Este é o ponto que mais pode inviabilizar o projeto se for deixado de lado. Quando se
> move dinheiro de real pra cripto — **ainda mais fazendo isso para outras empresas** —
> o Banco Central e a Receita têm regras. A parte técnica disso o sistema já está
> preparado pra encaixar; o que falta é **a decisão de como vamos nos posicionar**. É
> bem provável que a gente precise de um **advogado especializado** — então aqui a
> reunião decide a direção e define quem busca essa orientação.

### P3 — Quem confere a identidade do cliente final: a Lunium ou a empresa que usa a Lunium? 🔴 DECIDIR
**O que é:** o **KYC** é a verificação de quem é a pessoa que está comprando (documento,
selfie, etc.). Como vamos vender nossa estrutura também para **outras empresas** usarem,
a pergunta é: **quem faz essa checagem** — nós ou o cliente empresarial?

**Por que importa:** na prática, **quem mexe no dinheiro costuma ser o responsável**
perante o governo, mesmo que o cliente "pertença" à outra empresa. Isso muda como o
sistema é construído e quem assume o risco legal.

- **Decisão:** _______________________________________________________
- **Quem cuida:** ______________  **Até quando:** ____________

### P4 — Precisamos nos registrar como "empresa de ativos virtuais" (VASP)? 🔴⚠️ DESIGNAR (advogado)
**O que é:** o Banco Central regula empresas que compram/vendem cripto (chamadas VASP).
Precisamos saber **se** e **quando** temos que nos registrar, e como classificar
juridicamente o nosso fluxo (PIX entra → vira "real digital" → fica numa carteira nossa).

**Por que importa:** operar fora dessa regra pode gerar multa ou ordem de parar.

- **Quem vai buscar a orientação (advogado/consultoria):** ___________________  **Prazo:** _____

### P5 — Regras anti-lavagem de dinheiro: o que precisamos cumprir? 🔴 DECIDIR + DESIGNAR
**O que é:** a lei obriga a (a) **checar se o cliente está em listas de pessoas
proibidas** (criminosos, sancionados, e os chamados "politicamente expostos" — PEP),
(b) **monitorar transações suspeitas** e (c) **reportar ao COAF** (o órgão do governo
que recebe esses avisos). Existem empresas que fazem essas checagens pra gente.

**Por que importa:** é obrigação legal; sem isso a operação fica exposta.

- **Direção decidida:** ________________________________________________
- **Empresa que fará as checagens:** _____________  **Quem avalia opções:** _________

### P6 — Quando enviamos a cripto, precisamos checar para quem estamos mandando? ⚠️ DECIDIR
**O que é:** ao enviar a cripto pra carteira do cliente, há uma regra internacional
(apelidada de "travel-rule") que, em certos casos, exige saber/registrar o destino.

**Por que importa:** mandar cripto pra um endereço qualquer sem nenhuma checagem é
justamente o que essas regras vigiam.

- **Decisão (vamos checar o endereço de destino? em quais casos?):** _______________________

### P7 — Obrigações com a Receita (impostos e declarações) 🔎 DESIGNAR (contador)
**O que é:** operação com cripto tem declarações obrigatórias à Receita Federal.

- **Responsável (contador):** ______________  **Prazo:** ____________

[PAGEBREAK]

## Bloco 3 — Preço e contas (quanto a gente ganha em cada venda)

### P8 — Quanto a Lunium ganha por operação, e como esse ganho é cobrado? 💰 DECIDIR
**O que é:** em cada compra do cliente, a gente embute um ganho no preço. Há três
formas de cobrar (exemplo com uma compra de R$ 100):

- **Percentual sobre o valor** — ex.: 2% → ganhamos R$ 2.
- **Margem no preço da cripto** ("spread") — vendemos a cripto um pouco mais cara do que
  compramos; a diferença é o ganho. O cliente vê só o preço final.
- **Valor fixo + percentual** — ex.: R$ 1 fixo + 1,5%.

O ganho fica **embutido no preço que o cliente vê na hora de comprar** (transparente,
sem taxa "por fora").

- **Forma escolhida:** ____________________  **Número:** ____________________

### P9 — Antes de fixar o ganho (P8), alguém precisa somar TODOS os custos 💰⚠️ DESIGNAR
**O que é:** o nosso produto tem custo dos **dois lados**: do lado do dinheiro entrando
(taxas do PIX/Eulen) e do lado da cripto saindo (taxa da corretora ~0,05%, taxa de envio
da cripto pela rede, e o custo de **reabastecer o estoque** — quando convertemos o que
recebemos de volta pra repor a corretora, o que tem suas próprias taxas). Tem ainda o
**risco de o preço da cripto mudar** entre a hora que cotamos e a hora que compramos.

**Por que importa:** **se a taxa (P8) for menor que a soma desses custos, a gente perde
dinheiro em cada venda sem perceber.** Hoje boa parte desses custos ainda não está na
conta. Por isso, antes de cravar o número, alguém precisa montar essa "planilha de
custo por operação".

- **Quem monta a planilha de custos:** ____________  **Prazo:** _________

### P10 — Qual o valor mínimo e o máximo por operação? 💰🔎 DECIDIR (depende dos dados de P19/P20)
**O que é:** o **mínimo não é uma escolha livre** — ele é o **maior** entre três pisos:
o valor mínimo de PIX que a Eulen aceita, o valor mínimo de compra na corretora, e o
valor mínimo pra enviar aquela cripto pela rede (varia por moeda). O nosso mínimo tem
que ser o maior desses três. O **máximo** é uma escolha de risco (quanto aceitamos numa
única operação).

- **Mínimo (depois dos dados):** ____________  **Máximo:** ____________

### P11 — Por quanto tempo a cotação fica "travada" para o cliente? 💰 DECIDIR
**O que é:** quando o cliente vê "R$ 100 = X de Bitcoin", esse preço vale por um tempo
até ele pagar. Esse prazo é a "validade da cotação".

**Por que importa:** se o preço da cripto **subir** entre a cotação e o pagamento, **quem
paga a diferença é a Lunium**. Prazo curto = menos risco pra gente, mas o cliente tem
menos tempo. Prazo longo = melhor pro cliente, mais risco pra gente.

- **Decisão (ex.: 2 min, 5 min, 15 min…):** ____________

---

## Bloco 4 — Estoque e imprevistos

### P12 — Como evitar "vender o que não temos em estoque"? ⚠️ DECIDIR
**O que é:** imagine que temos R$ 1.000 de Bitcoin em estoque. Dois clientes pedem
R$ 1.000 cada **no mesmo instante**. Os dois veem que "tem estoque" e pagam — mas só dá
pra entregar pra um. O outro fica "pago e sem receber" até a gente repor.

O sistema já tem um mecanismo de **trava** que segura o estoque assim que alguém aceita a
cotação (evita prometer o mesmo estoque duas vezes). A decisão é: **ligamos essa trava já
no começo**, e com qual **folga de segurança** (uma margem pra não raspar o estoque)?

- **Decisão (ligar a trava? qual folga?):** ____________________________________

### P13 — E quando a corretora não consegue comprar tudo de uma vez? ⚠️💰 DECIDIR
**O que é:** às vezes, no instante da compra, o mercado não tem cripto suficiente
naquele preço, e a corretora compra só uma parte do pedido (ex.: 92% do que pedimos).
O sistema tolera uma pequena diferença automaticamente. Se a diferença for grande, a
operação **fica "em análise" para uma pessoa decidir**.

**A decisão:** qual a regra padrão quando falta muito — **entregar a parte que deu**,
**comprar o restante** (a um preço novo, risco nosso) ou **devolver o dinheiro**?

- **Tolerância automática (ex.: até 10%):** ________  **Regra padrão acima disso:** ________________

### P14 — Quanto dinheiro deixar parado como estoque na corretora? 💰⚠️ DECIDIR
**O que é:** pra entregar cripto na hora, precisamos manter um valor **comprado e parado**
na corretora (o estoque). Esse valor **limita quanto a gente consegue vender por dia**:
quando o estoque acaba, alguém precisa **repor manualmente**, e isso leva um tempo.

**Por que importa:** estoque pequeno = a gente trava as vendas com frequência; estoque
grande = mais dinheiro "preso". Liga direto ao caixa do projeto (P21).

- **Estoque inicial (R$/US$):** ____________  **Vendas/dia que queremos suportar:** ____________

### P15 — Se algo der errado DEPOIS do cliente pagar, como devolvemos o dinheiro? ⚠️🔎 DECIDIR + DESIGNAR
**O que é:** em casos raros, o cliente paga e a entrega não acontece (faltou estoque, a
rede de envio caiu, etc.). Aí precisamos **devolver o dinheiro**. Como por enquanto não
construímos o caminho automático de devolução, precisamos saber: **a Eulen devolve o PIX
automaticamente** (pelo sistema dela) **ou a devolução é feita na mão**? E em quanto tempo
nos comprometemos a devolver?

- **Como devolve (automático pela Eulen ou manual):** ____________  **Prazo de devolução:** ________
- **Quem confirma isso com a Eulen:** ______________  **Até quando:** ____________

### P16 — Quando uma operação trava, qual a regra: tentar de novo ou devolver? ⚠️ DECIDIR
**O que é:** quando uma operação fica "em análise" (dinheiro do cliente parado), a pessoa
que cuida da operação precisa de uma **regra clara** pra decidir entre **tentar de novo**
ou **devolver** — pra não ficar no improviso.

- **Regra:** ______________________________________________________

[PAGEBREAK]

## Bloco 5 — Corretora e dados a confirmar

> Quase tudo aqui é **DESIGNAR**: depende de abrir conta, ler contrato ou pegar
> informação técnica. O importante é sair com **responsável + prazo** pra cada item.

### P17 — Confirmar que as "contas-filhas" da corretora servem pro nosso modelo ⚠️🔎 DESIGNAR (importante)
**O que é:** pra não misturar o dinheiro de todos os clientes numa conta só, a ideia é
criar **uma "sub-conta" (conta-filha) por cliente** dentro da nossa conta na MEXC.
Precisamos **confirmar com a MEXC** duas coisas: (1) que essas contas-filhas conseguem
**receber depósito e fazer saque sozinhas**; e (2) que o **contrato de uso da MEXC
permite** a gente guardar dinheiro de terceiros assim.

**Por que importa:** se o contrato não permitir, a corretora pode **congelar a conta** —
e aí estoque e operações em andamento ficam presos. É uma suposição central do projeto
que precisa ser checada **antes** de programar em cima dela.

- **Quem verifica:** ______________  **Prazo:** ____________  **Resultado:** ________

### P18 — Aceitamos depender de uma única corretora no começo? ⚠️ DECIDIR
**O que é:** no início, toda a saída de cripto passa por **uma corretora só** (a MEXC).
Trocar/ter uma segunda corretora fica pra depois.

**Por que importa:** se a MEXC **congelar nossa conta** ou tiver um problema, a operação
para. Precisamos aceitar esse risco conscientemente (e pensar numa mitigação simples).

- **Aceitamos? (sim/não + alguma mitigação):** _________________________________

### P19 — Pegar as informações técnicas da MEXC pra conectar o sistema 🔎 DESIGNAR
**O que é:** dados técnicos pra ligar nosso sistema à corretora (chaves de acesso,
endereços de conexão, limites de envio por moeda, etc.). Isso o time técnico extrai.

- **Quem extrai:** ______________  **Prazo:** ____________

### P20 — Abrir a conta nova da Eulen e pegar os dados dela 🔎 DESIGNAR
**O que é:** credenciais e informações técnicas da conta empresarial **nova** da Eulen
(como ela cria a cobrança PIX, como ela nos avisa do pagamento, os limites, etc.).

- **Quem abre/extrai:** ______________  **Prazo:** ____________

---

## Bloco 6 — Caixa do projeto

### P21 — Como dividir o dinheiro aportado e por quanto tempo ele dura 💰 DECIDIR
**O que é:** temos os aportes (Igor US$ 20 mil; CTO US$ 3 mil pra integrações). Preciso
saber **quanto vai virar estoque na corretora** (P14), **quanto vai pra montar o sistema
e as integrações**, e **quanto fica de reserva** — e por quanto tempo isso nos segura até
começar a operar.

- **Estoque:** ________  **Sistema/integrações:** ________  **Dura quantos meses:** ________

---

## Bloco 7 — Site e contas

### P22 — O site comprecripto.io está registrado, e qual o plano de "virada" dele? 🔎 DESIGNAR
**O que é:** confirmar que o domínio `comprecripto.io` está registrado e combinar quando/
como ele passa a apontar pro nosso sistema (a "virada", ou cutover).

- **Está registrado? (sim/não):** ________  **Responsável pela virada:** __________

### P23 — Quem abre cada conta e até quando 🔎 DESIGNAR
**O que é:** definir responsáveis e prazos pra abrir a conta da Eulen e a da MEXC (esta
com permissão de saque e contas-filhas).

- **Eulen — responsável/prazo:** ______________  **MEXC — responsável/prazo:** ______________

[PAGEBREAK]

## Apêndice A — O que o CTO resolve sozinho (vocês podem ignorar)

> São decisões **técnicas internas**, daquelas que eu resolvo enquanto programo. Estão
> aqui só pra deixar claro que **não dependem de vocês e não travam a reunião**. Não
> precisam ler — é mais pra mostrar que essa parte está coberta.

- Quais ferramentas e versões usar no sistema (banco de dados, linguagem, esteira de
  testes automáticos).
- Os tempos internos de espera e de nova tentativa quando algo falha (eu defino a partir
  dos limites reais das contas, assim que os dados de P19/P20 chegarem).
- Como o sistema garante que nada se perde nem é cobrado em dobro internamente.
- Detalhes de organização do código e de como conectar cada serviço.

---

## Apêndice B — Checklist: "fechamos o planejamento, dá pra programar"

- [ ] **P1 e P2** decididos → contas da MEXC e da Eulen em processo de abertura.
- [ ] **Bloco 2 (regras do governo)** com a direção decidida e o advogado/contador acionados.
- [ ] **P8 a P11** definidos (ou ao menos a planilha de custos de P9 com responsável) → fecha a parte de preço.
- [ ] **P12 a P16** definidos → fecha estoque, imprevistos e devolução.
- [ ] **P17 a P20** com responsável e prazo → os dados técnicos vão chegando.
- [ ] **P21** define o caixa e o tamanho do estoque.
- [ ] Cada decisão foi **registrada** no projeto, e a **ata** da reunião aponta pra elas.

**Assim que o item P1 (conta da corretora) e o Bloco 2 (regras do governo) estiverem
encaminhados, e P8 a P16 decididos, eu começo a programar o sistema. As partes que
dependem dos dados das contas (P17 a P20) eu adianto com simulações até os dados reais
chegarem — ou seja, a programação não fica parada esperando.**
