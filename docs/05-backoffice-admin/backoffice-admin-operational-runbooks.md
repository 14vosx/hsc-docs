# Operational Runbooks

## Navegação rápida

- [Home da documentação](../README.md)
- [Backoffice Admin](./README.md)
- [Master Index](../00-governance/99-master-index.md)

---
## Objetivo

Documentar os runbooks operacionais reais do Backoffice Admin, cobrindo desenvolvimento local e, principalmente, o fluxo publicado de login administrativo por magic link, incluindo troubleshooting de CORS, cookie, SMTP, callback e sessão não validada.

Este documento existe para registrar, de forma estável e auditável:

- como operar o Backoffice localmente
- como validar o login administrativo publicado
- como solicitar magic link e consumi-lo corretamente
- como confirmar a navegação para `/dashboard`
- como diagnosticar falhas comuns do fluxo real

---

## Escopo

Este documento cobre:

- runbook local mínimo do Backoffice
- runbook publicado do login administrativo
- validação de `/login`
- solicitação de magic link
- recebimento do email Hostinger
- consumo do link
- validação de `/auth/callback`
- validação da navegação para `/dashboard`
- troubleshooting de CORS, cookie, SMTP, callback e sessão não validada

Este documento não cobre em profundidade:

- deploy final do frontend administrativo
- runbooks específicos de domínio (`news`, `seasons`, `events`)
- troubleshooting profundo do host Lightsail

---

## Estado atual

O Backoffice publicado já sustenta o fluxo real de login administrativo fim a fim.

O estado reconciliado inclui:

- Backoffice publicado em subdomínio próprio
- tela `/login` funcional
- request real de magic link
- email real via Hostinger SMTP
- consume do link pela Auth API publicada
- callback administrativo em `/auth/callback`
- revalidação da sessão via `GET /auth/session`
- navegação automática para `/dashboard`

---

## Relações com outros documentos

Este documento deve ser lido junto com:

- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-references-inventory.md`

---

## Runbook local mínimo

### Subir a Auth API local

```bash
ENV_FILE=.env.local ./ops/dev.sh
```

Resultado esperado:

- MariaDB local sobe
- API sobe em `127.0.0.1:3000`
- log mostra `Database readiness check passed.`

### Subir o Backoffice local

```bash
npm start
```

Pré-condição:

- `ng serve` com proxy configurado para a Auth API local

### Smoke local mínimo

```bash
curl -i http://127.0.0.1:3000/health
```

Esperado:

- `200 OK`

---

## Runbook publicado do login administrativo

### Passo 1 — validar `/login`

Abrir:

```text
https://backoffice.haxixesmokeclub.com/login
```

Resultado esperado:

- tela de login carregada
- campo de email disponível
- ação de envio de magic link disponível

### Passo 2 — solicitar magic link

Informar um email administrativo elegível, por exemplo:

```text
no-reply@haxixesmokeclub.com
```

Acionar:

- botão de envio do magic link

Resultado esperado:

- request para `POST /auth/magic-link/request`
- resposta `200 OK`
- mensagem genérica de continuidade na UI

### Passo 3 — receber email Hostinger

Abrir a caixa do Hostinger associada ao remetente/operador elegível.

Resultado esperado:

- email com assunto do login admin
- link apontando para `https://auth-api.haxixesmokeclub.com/auth/magic-link/consume?token=...`

### Passo 4 — consumir o link

Clicar no link do email.

Resultado esperado:

- navegador chega ao consume da Auth API
- backend cria a sessão e emite cookie
- navegador é redirecionado para `/auth/callback?status=ok`

### Passo 5 — validar `/auth/callback`

Resultado esperado:

- callback carrega sem 404
- callback tenta revalidar a sessão
- callback não deve exigir `F5` após o retry curto reconciliado

### Passo 6 — validar navegação para `/dashboard`

Resultado esperado:

- operador entra automaticamente em `/dashboard`
- shell administrativo é exibido como autenticado

---

## Checks práticos de rede e browser

### Request do login

No DevTools, validar:

- `POST https://auth-api.haxixesmokeclub.com/auth/magic-link/request`
- sem erro de CORS
- resposta JSON válida

### Consume do link

Validar:

- status `302 Found`
- `Set-Cookie: hsc_admin_session=...`
- redirect para `/auth/callback?status=ok`

### Revalidação da sessão

Validar:

- request para `GET /auth/session`
- request com credenciais
- resposta `200` com `authenticated: true` após login consolidado

---

## Troubleshooting — CORS

### Sintoma

A UI mostra falha ao solicitar magic link e o DevTools acusa bloqueio de origem.

### Verificações

- `ALLOWED_ORIGINS` inclui `https://backoffice.haxixesmokeclub.com`
- backend responde com CORS compatível com credenciais
- frontend chama com `withCredentials: true`
- request vai para `auth-api.haxixesmokeclub.com`, não para o próprio host estático do Backoffice

---

## Troubleshooting — cookie

### Sintoma

O consume retorna `status=ok`, mas `GET /auth/session` logo depois não autentica.

### Verificações

- resposta do consume inclui `Set-Cookie`
- política do cookie é compatível com produção HTTPS cross-subdomain
- request seguinte para `/auth/session` realmente envia o cookie
- callback aplica retry curto antes de concluir erro

---

## Troubleshooting — SMTP

### Sintoma

`POST /auth/magic-link/request` responde `200`, mas email não chega.

### Verificações

- usuário admin elegível existe no banco
- envs SMTP estão corretos no `.env` do servidor
- serviço foi reiniciado após mudança de env
- logs do `hsc-auth-api` mostram `delivered to=...` ou falha SMTP explícita

### Dependência operacional real

- Hostinger SMTP
- conta `no-reply@haxixesmokeclub.com`

---

## Troubleshooting — callback

### Sintoma

Após clicar no link, o operador cai em `/auth/callback?error=...`

### Códigos relevantes

- `invalid_or_expired_link`
- `missing_token`
- `forbidden`
- `consume_failed`
- `db_not_ready`

### Interpretação operacional

- `invalid_or_expired_link`: token inválido, expirado ou já utilizado
- `consume_failed`: falha interna do backend durante consolidação do login
- `db_not_ready`: backend de pé, mas sem prontidão funcional do banco

---

## Troubleshooting — sessão não validada

### Sintoma

O operador chega em `status=ok`, mas a tela ainda mostra “sessão não pôde ser validada”.

### Verificações

- `GET /auth/session` saiu com `withCredentials: true`
- cookie foi emitido e persistido
- callback com retry curto está publicado no build correto
- request para `/auth/session` retorna `200` autenticado, não `401`

---

## Comandos úteis de backend para troubleshooting

### Logs em produção

```bash
sudo journalctl -u hsc-auth-api -f --no-pager
```

### Health publicado

```bash
curl -i https://auth-api.haxixesmokeclub.com/health
```

### Sessão sem cookie

```bash
curl -i https://auth-api.haxixesmokeclub.com/auth/session
```

### Schema técnico protegido

```bash
curl -i -H "X-Admin-Key: <ADMIN_KEY>" https://auth-api.haxixesmokeclub.com/admin/schema
```

---

## Critério de pronto deste documento

Este runbook pode ser considerado reconciliado quando:

- o fluxo publicado `/login` → email → `consume` → `/auth/callback` → `/dashboard` estiver descrito
- troubleshooting de CORS, cookie, SMTP, callback e sessão não validada estiver coberto
- o papel da Auth API como autoridade final estiver implícito em todo o fluxo

Neste checkpoint, esse critério está atendido.
