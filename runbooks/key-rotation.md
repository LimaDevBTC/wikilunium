# Runbook: Rotação de chaves e credenciais

> Procedimento para rotacionar segredos do sistema sem downtime.
> Seguir em routine (cadência) e em emergência (suspeita de comprometimento).
>
> **STATUS: atualizado em 2026-06-23**

---

## 1. Inventário de segredos

| Segredo | Onde vive | Impacto se vazar | Cadência routine |
|---|---|---|---|
| `MEXC_API_KEY` + `MEXC_SECRET_KEY` | Cofre → `.env` no VPS | Saque de USDT para endereço allowlistado | 90 dias |
| `SMARTPAY_API_KEY` | Cofre → `.env` no VPS | Criação de payout orders | 90 dias |
| `SMARTPAY_WEBHOOK_SECRET` | Cofre → `.env` no VPS | Aceitação de webhooks forjados | 90 dias |
| `ADMIN_TOKEN` | Cofre → `.env` no VPS | Leitura e ações administrativas | 90 dias |
| `CLIENT_API_TOKEN` | Cofre → `.env` no VPS + cliente | Criação de cotações/aceitações | 90 dias ou a pedido do cliente |
| `TELEGRAM_BOT_TOKEN` | Cofre → `.env` no VPS | Alertas não chegam / spoof | 180 dias |
| `DATABASE_URL` (senha Postgres) | Cofre → `.env` no VPS | Acesso direto ao banco | 180 dias |
| `REDIS_URL` (senha Redis, se configurada) | Cofre → `.env` no VPS | Acesso à fila | 180 dias |

**Regra:** nenhum segredo vai para código, histórico git, Slack ou mensagem. Qualquer vazamento acidental = rotacionar imediatamente.

---

## 2. Procedimento geral (sem downtime)

O padrão é **nova chave primeiro, trocar depois, revogar a antiga**:

```
1. Gerar nova credencial no provedor/sistema
2. Adicionar a nova credencial ao cofre
3. Atualizar .env no VPS com a nova credencial
4. Reiniciar a lunium-api (sem downtime se usar pm2 reload / docker restart)
5. Confirmar que a nova credencial está funcionando (smoke test)
6. Revogar a credencial antiga no provedor
```

Nunca revogar antes de confirmar que a nova está ativa. Uma janela de ~5 min com ambas ativas é aceitável e segura.

---

## 3. Por credencial

### 3.1 — MEXC (`MEXC_API_KEY` + `MEXC_SECRET_KEY`)

1. Acessar MEXC → Gerenciamento de API → Criar nova chave.
2. Configurar permissões **mínimas necessárias**: Leitura + Trade + Withdrawal (apenas para o endereço SmartPay na allowlist).
3. **Antes de ativar a nova chave:** confirmar que o endereço de saque SmartPay está na allowlist da nova chave também.
4. Atualizar `MEXC_API_KEY` e `MEXC_SECRET_KEY` no `.env` do VPS.
5. Reiniciar a lunium-api.
6. Smoke test: `GET /admin/reconciliation` deve retornar sem erros de autenticação MEXC.
7. Revogar a chave antiga no painel MEXC.

> **Atenção:** a allowlist de endereço de saque é por chave de API na MEXC. Ao criar nova chave, reconfigurar a allowlist imediatamente — não herda da chave anterior.

---

### 3.2 — SmartPay (`SMARTPAY_API_KEY` + `SMARTPAY_WEBHOOK_SECRET`)

1. Acessar painel SmartPay → Configurações de API → Gerar nova chave.
2. Gerar novo `WEBHOOK_SECRET` no mesmo painel.
3. Atualizar as duas vars no `.env` do VPS.
4. Reiniciar a lunium-api.
5. Smoke test: disparo de teste de webhook da SmartPay deve retornar 200.
6. Revogar credenciais antigas.

> **Atenção:** trocar `SMARTPAY_WEBHOOK_SECRET` sem reiniciar a API causa rejeição de todos os webhooks da SmartPay. Planejar para horário de baixo volume.

---

### 3.3 — `ADMIN_TOKEN`

1. Gerar novo token: `openssl rand -base64 32`
2. Atualizar `ADMIN_TOKEN` no `.env` do VPS.
3. Reiniciar a lunium-api.
4. Confirmar acesso: `GET /admin/reconciliation` com o novo token.
5. Comunicar o novo token ao operador (Guilherme) via canal seguro.

---

### 3.4 — `CLIENT_API_TOKEN`

1. Gerar novo token: `openssl rand -base64 32`
2. Coordenar com o cliente (`vendacripto-app`): definir horário de troca.
3. **Janela de migração:** atualizar o `.env` do VPS e reiniciar a API com a nova chave. O cliente deve atualizar em paralelo no mesmo intervalo (~5 min de tolerância).
4. Se o cliente não conseguir atualizar a tempo: manter a chave antiga temporariamente e repetir o processo.

> Não há suporte a múltiplos tokens simultâneos no MVP. Em caso de múltiplos clientes futuros, migrar para tabela de API keys no banco.

---

### 3.5 — `TELEGRAM_BOT_TOKEN`

1. No BotFather do Telegram: `/mybots` → selecionar o bot → API Token → Revoke current token → Generate new token.
2. Atualizar `TELEGRAM_BOT_TOKEN` no `.env` do VPS.
3. Reiniciar a lunium-api.
4. Confirmar: forçar um log de alerta e verificar chegada no grupo/canal Telegram.

---

### 3.6 — Senha do Postgres (`DATABASE_URL`)

> Maior cuidado: a lunium-api tem conexão persistente ao banco. Troca exige restart.

1. Criar nova senha forte: `openssl rand -base64 24`
2. No Postgres: `ALTER USER lunium WITH PASSWORD 'nova_senha';`
3. Atualizar `DATABASE_URL` no `.env` do VPS.
4. Reiniciar a lunium-api (as conexões existentes morrem; novas usam a nova senha).
5. Confirmar que a API reiniciou e está conectando ao banco.

> Se o Postgres estiver no mesmo VPS via Docker Compose: `docker compose exec postgres psql -U lunium -c "ALTER USER lunium WITH PASSWORD 'nova_senha';"`

---

## 4. Rotação de emergência (comprometimento suspeito)

Se houver suspeita de vazamento de qualquer chave:

1. **Revogar imediatamente** no provedor (sem esperar a janela de migração).
2. **Ativar o circuit-breaker:** `CASHOUT_ENABLED=false` no `.env` e reiniciar. Isso fecha novas cotações enquanto a situação é avaliada.
3. Verificar se houve saques não autorizados (MEXC) ou payout orders indevidas (SmartPay).
4. Gerar nova credencial e seguir o procedimento da §3 correspondente.
5. Reativar (`CASHOUT_ENABLED=true`) após confirmar que tudo está limpo.
6. Registrar o incidente (o quê, quando, como identificado, impacto, ação).

---

## 5. Verificação pós-rotação

Após qualquer rotação:
- [ ] `GET /admin/reconciliation` retorna 200 sem erros de auth
- [ ] Nenhuma operação travada por erro de credencial nos logs
- [ ] AlertService dispara um teste e chega no Telegram (se TELEGRAM configurado)
- [ ] Credencial antiga revogada no provedor
- [ ] Cofre atualizado com a nova credencial e data de rotação
