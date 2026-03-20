# Auth API Operations

## Objetivo

Documentar a operação funcional da Auth API no contexto AWS Lightsail, registrando as superfícies reais de autenticação administrativa, os invariantes do fluxo de magic link, a política de CORS, o papel do backend dinâmico no ecossistema HSC e os smoke tests mínimos que sustentam o Backoffice Admin publicado.

Este documento existe para registrar, de forma estável e auditável:

- o papel operacional da Auth API como backend dinâmico central do HSC
- as superfícies públicas e administrativas realmente publicadas
- o modelo administrativo session-first publicado
- o fluxo real de magic link administrativo
- o contrato operacional de `GET /auth/session`
- a relação entre sessão, cookie, callback e dashboard
- a política explícita de CORS para o Backoffice Admin
- o caminho break-glass administrativo e seus limites
- os invariantes operacionais do login administrativo
- os smoke tests mínimos após release e deploy

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Infra AWS Lightsail](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Runtime e infraestrutura imediata
- [Architecture Runtime](./infra-aws-lightsail-architecture-runtime.md)
- [Nginx Reverse Proxy](./nginx-reverse-proxy.md)
- [Node Systemd](./node-systemd.md)
- [MariaDB Local](./mariadb-local.md)
- [Deploy / Release / Rollback](./deploy-release-rollback.md)

### Consumo administrativo e contratos
- [Backoffice Admin](../05-backoffice-admin/README.md)
- [Admin API Contracts](../05-backoffice-admin/admin-api-contracts.md)
- [Auth, RBAC and Guards](../05-backoffice-admin/auth-rbac-and-guards.md)
- [Frontend Structure](../05-backoffice-admin/backoffice-admin-frontend-structure.md)

### Operação e suporte
- [Operational Runbooks](./infra-aws-lightsail-operational-runbooks.md)
- [Observability and Troubleshooting](./infra-aws-lightsail-observability-troubleshooting.md)
- [Documentation System](../00-governance/documentation-system.md)

---

## Escopo

Este documento cobre:

- o papel operacional da Auth API
- superfícies públicas operacionais
- superfícies administrativas reais
- autenticação administrativa baseada em sessão e magic link
- contratos de `POST /auth/magic-link/request`, `GET /auth/magic-link/consume` e `GET /auth/session`
- bootstrap local controlado de sessão admin
- caminho break-glass administrativo
- política de CORS e consumers autorizados
- dependências de SMTP transacional
- invariantes operacionais do fluxo session-first
- smoke tests de operação recorrente

Este documento não cobre em profundidade:

- configuração detalhada do Nginx
- unit file completo do systemd
- backup e restore do banco
- modelagem funcional profunda de `news`, `seasons` ou `events`
- operação detalhada do Backoffice frontend
- troubleshooting profundo de host/DNS/TLS

Esses assuntos vivem em documentos próprios do contexto.

---

## Estado atual

A Auth API opera como backend dinâmico central do HSC no contexto AWS Lightsail.

O estado operacional reconciliado desta camada inclui:

- execução em Node.js via systemd
- exposição pública por Nginx com TLS e reverse proxy
- persistência em MariaDB local
- superfícies públicas de conteúdo (`health`, `content/news`, `content/seasons`)
- superfícies administrativas protegidas (`admin/schema`, mutações de `news` e `seasons`)
- autenticação administrativa session-first publicada
- emissão real de magic link administrativo por SMTP Hostinger
- introspecção administrativa por `GET /auth/session`
- callback administrativo do Backoffice publicado em subdomínio próprio
- política de CORS por allowlist explícita de origens
- cookie administrativo cross-subdomain com política compatível com produção
- `schema.js` tratado como compatibilidade legada e não como mecanismo principal de evolução do banco
- migrations SQL explícitas executadas no deploy antes do restart do serviço

A Auth API sustenta a metade dinâmica do ecossistema HSC, enquanto o host Hostinger continua sustentando jogo, ETL e portal público estático.

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase, são:

- release line reconciliada da Auth API até `v0.4.7`
- deploy real em Lightsail com `npm run db:migrate` antes do restart
- tabelas `sessions`, `magic_links` e `schema_migrations` já reconciliadas em produção
- login administrativo fim a fim publicado e validado
- Backoffice publicado em `backoffice.haxixesmokeclub.com`
- SMTP Hostinger ativo para `no-reply@haxixesmokeclub.com`
- impl-log `2026-03-19-auth-admin-magic-link-and-sql-migrations-cutover.md`
- runbooks e contratos reconciliados dos contextos `04` e `05`

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-architecture-runtime.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-observability-troubleshooting.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-references-inventory.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/95-impl-log/2026-03-19-auth-admin-magic-link-and-sql-migrations-cutover.md`

Este documento descreve a operação funcional real da Auth API publicada. Ele não substitui a documentação do host, do banco ou do frontend administrativo.

---

## Papel operacional da Auth API

Do ponto de vista do ecossistema HSC, a Auth API não é apenas uma API de conteúdo.

A camada responde por:

- conteúdo dinâmico público relevante ao ecossistema
- superfícies administrativas protegidas
- autenticação e identidade administrativa
- emissão e consumo de magic links administrativos
- resolução da sessão administrativa atual
- validação de autorização em mutações sensíveis
- trilha administrativa persistida
- suporte dinâmico ao Backoffice Admin

Regra importante:

- a Auth API é a autoridade final de autenticação e autorização administrativa
- o Backoffice não deve assumir estado autenticado sem confirmação do backend

---

## Consumers autorizados

Os consumers operacionais relevantes desta Auth API, no estado atual, são:

- portal e inspeções operacionais via o próprio host Lightsail
- operadores humanos via `curl`/browser para smoke e troubleshooting
- `backoffice.haxixesmokeclub.com` como consumidor legítimo da sessão administrativa publicada
- ambiente local do Backoffice via proxy (`localhost`) durante desenvolvimento

Consumer administrativo publicado explicitamente autorizado:

- `https://backoffice.haxixesmokeclub.com`

Consumers locais explicitamente autorizados:

- `http://localhost:5173`
- variações locais controladas do fluxo de desenvolvimento quando configuradas no allowlist

---

## Política de CORS

A Auth API utiliza allowlist explícita de origens.

Estado reconciliado do modelo:

- não existe política aberta por wildcard para superfícies administrativas
- o Backoffice publicado é origem explicitamente permitida
- o ambiente local pode ser permitido de forma controlada para desenvolvimento
- requests com credenciais exigem compatibilidade entre origem permitida e política de cookie

Invariantes operacionais:

- requests administrativos cross-origin devem ser servidos com allowlist explícita
- o fluxo published Backoffice ↔ Auth API depende de `credentials: true` no backend e `withCredentials: true` no frontend
- mudanças em CORS devem ser tratadas como mudança operacional relevante, nunca como ajuste cosmético

---

## Session-first publicado

O modelo administrativo publicado é session-first.

Isso significa:

- o fluxo normal de operação admin não depende de segredo bruto por request
- a identidade administrativa é resolvida por sessão persistida
- o estado autenticado do operador é introspectado por `GET /auth/session`
- o Backoffice só considera o operador autenticado após confirmação explícita do backend

### Componentes desse modelo

- tabela `sessions` com `token_hash`, `expires_at`, `revoked_at`, `created_at`, `updated_at`
- cookie `hsc_admin_session`
- `GET /auth/session` como contrato real de introspecção
- `POST /auth/magic-link/request` como pedido de login
- `GET /auth/magic-link/consume` como consolidação da sessão
- callback do Backoffice em `/auth/callback`

### Caminho break-glass

O contexto mantém caminho break-glass administrativo para contingência e operação técnica controlada.

Estado atual reconciliado:

- `X-Admin-Key` continua relevante para superfícies técnicas e smoke controlado
- `admin/schema` continua útil para validação operacional e troubleshooting
- o uso administrativo normal do Backoffice não deve depender desse caminho

Regra importante:

- session-first é o caminho normal de operação
- break-glass existe para contingência, inspeção técnica e operação controlada

---

## Superfícies públicas operacionais

As superfícies públicas operacionais conhecidas incluem:

### `GET /health`

Função:

- validar disponibilidade do runtime
- expor estado básico da aplicação
- refletir prontidão do banco por `db.ready`
- expor metadados mínimos de CORS úteis para troubleshooting

Interpretação operacional:

- `ok: true` + `db.ready: true` significa backend pronto para servir tráfego dinâmico relevante
- `ok: true` + `db.ready: false` significa processo web de pé, mas backend ainda não pronto para tráfego que depende do banco

### `GET /content/news`
### `GET /content/news/:slug`

Função:

- expor conteúdo público de `news`
- servir o ecossistema dinâmico do HSC

### `GET /content/seasons`
### `GET /content/seasons/active`
### `GET /content/seasons/:slug`

Função:

- expor catálogo e season ativa
- sustentar integração com outras camadas do ecossistema

---

## Superfícies administrativas reais

Superfícies administrativas relevantes no estado atual:

- `GET /admin/schema`
- endpoints administrativos de `news`
- endpoints administrativos de `seasons`
- `GET /auth/session`
- `POST /auth/magic-link/request`
- `GET /auth/magic-link/consume`
- `POST /auth/dev/bootstrap-session` (somente dev/local)

Regra importante:

- superfícies administrativas não pertencem ao portal público
- toda mutação administrativa deve ser tratada como operação autenticada e auditável

---

## Contrato operacional de `POST /auth/magic-link/request`

### Papel

Solicitar um magic link administrativo para um email elegível.

### Consumo esperado

- o Backoffice publicado chama esta rota a partir de `/login`
- o frontend envia `withCredentials: true`
- o backend responde com mensagem genérica para evitar enumeração de usuários

### Resposta funcional esperada

Mesmo quando o operador vê `200 OK`, isso não significa que o email foi necessariamente entregue para qualquer endereço arbitrário.

A resposta é propositalmente genérica.

Comportamento esperado:

- se o email é elegível e o delivery está funcional, o email é enviado
- se o email não é elegível, a resposta continua genérica
- em caso de falha interna de delivery, a resposta do fluxo administrativo deve ser tratada por logs e troubleshooting, sem abrir enumeração do diretório de usuários

### Dependências

- usuário admin elegível existente no banco
- `magicLinkDelivery.js` funcional
- SMTP Hostinger configurado no `.env` de produção
- `AUTH_API_PUBLIC_URL`, `BACKOFFICE_URL` e `MAGIC_LINK_CALLBACK_PATH` corretos

---

## Contrato operacional de `GET /auth/magic-link/consume`

### Papel

Consumir um token one-time de magic link, criar a sessão administrativa e redirecionar o operador para o callback do Backoffice.

### Comportamento esperado

- o token é one-time use
- o backend valida expiração e uso prévio
- quando válido, cria a sessão administrativa
- emite `Set-Cookie` para `hsc_admin_session`
- redireciona para `https://backoffice.haxixesmokeclub.com/auth/callback?status=ok`

### Falhas esperadas

Quando o consumo não pode prosseguir, o backend redireciona para callback com erro explícito.

Erros observados/operacionais relevantes:

- `missing_token`
- `invalid_or_expired_link`
- `forbidden`
- `consume_failed`

### Invariantes

- o token não pode ser reutilizado após consumo bem-sucedido
- a criação da sessão depende de schema reconciliado de `sessions`
- o fluxo publicado depende de cookie compatível com cross-subdomain

---

## Contrato operacional de `GET /auth/session`

### Papel

Resolver a sessão administrativa atual do operador.

### Comportamento esperado

Com sessão válida:

- `200 OK`
- `authenticated: true`
- `user`
- `role`

Sem sessão válida:

- `401 Unauthorized`
- `authenticated: false`

### Papel no ecossistema

- é o contrato session-first real do Backoffice
- guards, shell, callback e refresh dependem dele
- o frontend não deve sintetizar esse estado sem introspecção real

### Dependências operacionais

- cookie `hsc_admin_session` presente e reaproveitado pelo navegador
- `withCredentials: true` no frontend
- política de cookie compatível com ambiente HTTPS cross-subdomain

---

## Cookie administrativo publicado

O cookie administrativo publicado foi reconciliado para suportar o fluxo entre subdomínios.

Em produção HTTPS, o comportamento esperado é compatível com:

- `HttpOnly`
- `Secure`
- `SameSite=None`
- `Path=/`
- `Max-Age` alinhado ao TTL da sessão

Em desenvolvimento HTTP local, a política pode ser mais permissiva para evitar quebra do fluxo local.

Regra importante:

- o cookie administrativo não é um detalhe de frontend
- ele é parte crítica do contrato publicado entre Auth API e Backoffice

---

## SMTP transacional administrativo

O delivery real do magic link administrativo utiliza SMTP Hostinger.

Dependência operacional reconciliada:

- caixa `no-reply@haxixesmokeclub.com`
- `SMTP_HOST=smtp.hostinger.com`
- `SMTP_PORT=465`
- `SMTP_SECURE=true`
- `SMTP_USER=no-reply@haxixesmokeclub.com`
- `SMTP_PASS` via `.env` do servidor
- `MAGIC_LINK_FROM_EMAIL=no-reply@haxixesmokeclub.com`

Invariantes:

- restart do serviço é necessário após alteração de `.env`
- runtime não deve capturar valores SMTP cedo demais; leitura em tempo de uso foi reconciliada
- falha de SMTP deve ser observável via logs do serviço

---

## Invariantes operacionais do magic link

O fluxo real de magic link administrativo depende dos seguintes invariantes:

1. o email admin precisa existir e estar elegível no banco
2. o delivery SMTP precisa estar configurado e funcional
3. `AUTH_API_PUBLIC_URL` precisa apontar para a Auth API publicada
4. `BACKOFFICE_URL` precisa apontar para o Backoffice publicado
5. o token precisa ser one-time use
6. a tabela `sessions` precisa ter shape reconciliado
7. o cookie precisa ser emitido com política compatível com produção
8. o callback do Backoffice precisa revalidar a sessão
9. o frontend precisa aceitar um pequeno retry curto após `status=ok`

Se qualquer um desses pontos falhar, o sintoma pode aparecer como:

- email não chega
- `invalid_or_expired_link`
- `consume_failed`
- callback com “sessão não pôde ser validada”

---

## Fluxo operacional publicado do login admin

Fluxo funcional publicado validado:

1. operador abre `https://backoffice.haxixesmokeclub.com/login`
2. informa o email administrativo elegível
3. Backoffice chama `POST /auth/magic-link/request`
4. Auth API entrega email real via SMTP Hostinger
5. operador abre o email e clica no link
6. navegador consome `GET /auth/magic-link/consume?token=...`
7. Auth API cria sessão e emite cookie
8. backend redireciona para `/auth/callback?status=ok`
9. callback reexecuta `GET /auth/session`
10. Backoffice navega automaticamente para `/dashboard`

Esse fluxo já foi validado fim a fim em produção.

---

## Smoke tests mínimos de operação

### Health

```bash
curl -i https://auth-api.haxixesmokeclub.com/health
```

Esperado:

- `200 OK`
- `"ok":true`
- `"db":{"ready":true` ... `}`

### Conteúdo dinâmico

```bash
curl -i https://auth-api.haxixesmokeclub.com/content/news
curl -i https://auth-api.haxixesmokeclub.com/content/seasons
curl -i https://auth-api.haxixesmokeclub.com/content/seasons/active
```

### Superfície técnica protegida

```bash
curl -i -H "X-Admin-Key: <ADMIN_KEY>" https://auth-api.haxixesmokeclub.com/admin/schema
```

### Sessão não autenticada

```bash
curl -i https://auth-api.haxixesmokeclub.com/auth/session
```

Esperado sem cookie válido:

- `401 Unauthorized`
- `{"authenticated":false}`

---

## Troubleshooting operacional resumido

### Sintoma: `POST /auth/magic-link/request` retorna 200 mas email não chega

Verificar:

- usuário admin elegível existe no banco
- envs SMTP estão carregados no processo atual
- serviço foi reiniciado após alteração do `.env`
- logs mostram `delivered to=...` ou falha SMTP específica

### Sintoma: `consume_failed`

Verificar:

- shape real da tabela `sessions`
- reconciliação de schema e migrations aplicadas
- logs do serviço durante o clique no link

### Sintoma: callback recebe `status=ok` mas sessão não valida

Verificar:

- `Set-Cookie` presente no consume
- política de cookie cross-subdomain
- request subsequente para `/auth/session` enviando cookie
- retry curto no callback do Backoffice

### Sintoma: erro de CORS no Backoffice

Verificar:

- allowlist real no `.env`
- `credentials: true` no backend
- `withCredentials: true` no frontend
- origem publicada correta do Backoffice

---

## Critério de pronto deste documento

Este documento pode ser considerado reconciliado quando:

- os contratos reais de `request`, `consume` e `session` estiverem refletidos aqui
- o papel do Backoffice publicado estiver explícito
- a política de CORS estiver atualizada
- os invariantes do magic link estiverem listados
- o modelo session-first publicado estiver descrito sem ambiguidade

Neste checkpoint, esse critério está atendido.
