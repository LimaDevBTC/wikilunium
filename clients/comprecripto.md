# Cliente #001 — comprecripto.io

> Registro canônico do **primeiro cliente** da infraestrutura Lunium.

> **STATUS: stub — a preencher**

<!-- comprecripto.io é CLIENTE da infra Lunium, não o produto Lunium. Os specs de
     infra (domain/, providers/, contratos) permanecem client-agnostic. Ver
     ADR-005. -->

## Identidade

- **Cliente:** Cliente #001
- **Domínio (front):** `comprecripto.io`
- **App / repo:** `comprecripto-app` (Next.js) — **repo SEPARADO**, não vive aqui
- **Status:** em desenvolvimento

## Capabilities consumidas

- `on_ramp` (Eulen) e `exchange` (MEXC), via `lunium-api` headless.
- TODO: detalhar o que o cliente expõe ao usuário final.

## Integração

- Consome a `lunium-api` (headless).
- TODO: endpoints, credenciais/escopo do cliente, cutover de DNS para
  `comprecripto.io`.

## Contatos

- TODO.

## Notas

- TODO.
