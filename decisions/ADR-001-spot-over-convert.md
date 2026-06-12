# ADR-001 — Spot, não Convert

> Como o Lunium executa a compra de cripto na MEXC.

> **STATUS: rascunho — revisar**

- **Status:** Aceito (rascunho)
- **Data:** 2026-06-10
- **Decisores:** Guilherme, Mateus, CTO

## Contexto

&lt;TODO: a preencher a partir da ata e da análise de custos.&gt;

## Decisão

A MEXC **não expõe Convert na API**. A compra é feita via **ordem spot** no par
contra USDT — mais barata (maker 0% / taker 0,05%) que o spread do Convert usado
hoje. O **fill parcial** é tratado: a ordem "market" é modelada como **limit IOC**
com tolerância de ~10%.

## Consequências

&lt;TODO: implica client spot tipado, tratamento de fill parcial, cálculo de
preço/slippage. Detalhar.&gt;

## Alternativas consideradas

&lt;TODO: Convert (descartado por custo/indisponibilidade na API). Detalhar.&gt;
