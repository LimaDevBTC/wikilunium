# Cliente #001 — vendacripto.io

> Registro canônico do **primeiro cliente** da infraestrutura Lunium.

<!-- vendacripto.io é CLIENTE da infra Lunium, não o produto Lunium. Os specs de
     infra (domain/, providers/, contratos) permanecem client-agnostic. Ver
     ADR-005. -->

## Identidade

- **Cliente:** Cliente #001
- **Domínio (front):** `vendacripto.io`
- **App / repo:** `vendacripto-app` (Next.js) — **repo SEPARADO**, não vive aqui
- **Status:** em desenvolvimento
- **Anteriormente:** `comprecripto.io` (nome alterado em 2026-06-22)

## Capabilities consumidas

- `off_ramp` (SmartPay) e `exchange` (MEXC), via `lunium-api` headless.
- MVP: cash-out (cliente vende cripto e recebe BRL via PIX).

## Integração

- Consome a `lunium-api` (headless).
- TODO: endpoints, credenciais/escopo do cliente, cutover de DNS para `vendacripto.io`.

## Contatos

- TODO.
