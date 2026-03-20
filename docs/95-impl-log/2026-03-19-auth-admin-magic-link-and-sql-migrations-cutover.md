# 2026-03-19 — Auth Admin Magic Link and SQL Migrations Cutover

## Objetivo

Registrar, de forma estável e auditável, o checkpoint técnico construído ao longo desta frente de trabalho, cobrindo:

- autenticação administrativa session-first
- magic link admin ponta a ponta
- publicação do Backoffice em subdomínio próprio
- endurecimento de CORS, cookie e sessão cross-subdomain
- entrega real de email via Hostinger SMTP
- correção de drift de schema em `sessions`
- transição do modelo legado de evolução de banco para SQL migrations por arquivo
- prova operacional do novo fluxo de migration em produção

Este impl-log existe para preservar a trilha incremental da mudança sem substituir os documentos canônicos dos contextos `04-infra-aws-lightsail` e `05-backoffice-admin`.

---

## Escopo

Este documento cobre:

- a sequência incremental de mudanças relevantes feitas nesta frente
- os principais problemas encontrados em local e produção
- as correções aplicadas no backend e no frontend administrativo
- as releases e deploys associados ao checkpoint
- o cutover operacional para `db/migrations/*.sql`
- o estado final validado em produção ao fim do checkpoint

Este documento não cobre em profundidade:

- toda a topologia do contexto AWS Lightsail
- a documentação estrutural completa do Backoffice
- a descrição completa de todos os arquivos do repositório
- backlog funcional futuro além do necessário para entender o checkpoint

Esses assuntos vivem nos documentos canônicos e inventários próprios.

---

## Estado inicial do checkpoint

No início desta frente, o ecossistema ainda estava em transição entre:

- superfícies administrativas novas na Auth API
- shell administrativo do Backoffice ainda conectando localmente
- dependência forte de `src/db/schema.js` como mecanismo de evolução de banco
- ausência de delivery SMTP real para magic link
- instabilidade no fluxo cross-subdomain entre `auth-api.haxixesmokeclub.com` e `backoffice.haxixesmokeclub.com`

Os sintomas mais relevantes observados no início incluíam:

- falhas de import e wiring em rotas de auth dev/local
- drift de schema local e em produção, especialmente na tabela `sessions`
- deploy falhando por indisponibilidade do MariaDB em produção
- CORS não pronto para o consumidor Backoffice publicado
- cookie de sessão não reutilizado corretamente no fluxo cross-subdomain
- ausência de um sistema real de migrations por arquivo

---

## Source of truth / evidências

As evidências diretas deste impl-log são:

- a trilha de execução desta frente técnica
- comandos executados e outputs validados ao longo da implementação
- releases e tags geradas na Auth API
- deploys e smokes realizados no Lightsail
- criação e validação do subdomínio `backoffice.haxixesmokeclub.com`
- validação do fluxo real de magic link com Hostinger SMTP
- validação do novo runner `scripts/migrate.js`
- validação da migration `0002_magic_links_add_used_at_index.sql` em produção

Artefatos diretamente envolvidos no checkpoint:

- `src/routes/auth/session.js`
- `src/routes/auth/request-magic-link.js`
- `src/routes/auth/consume-magic-link.js`
- `src/services/auth/magicLinkDelivery.js`
- `src/services/auth/magicLinkContract.js`
- `src/utils/sessionCookie.js`
- `src/db/adminSessions.js`
- `src/db/magicLinks.js`
- `src/db/schema.js`
- `src/db/bootstrap.js`
- `db/migrations/0000_schema_migrations_bootstrap.sql`
- `db/migrations/0001_baseline_v0_4_5.sql`
- `db/migrations/0002_magic_links_add_used_at_index.sql`
- `scripts/migrate.js`
- `ops/deploy-auth.sh`
- `ops/release.sh`
- `src/app/core/auth/auth-api.service.ts`
- `src/app/features/auth/pages/auth-callback-page/auth-callback-page.component.ts`

---

## Relações com outros documentos

Este impl-log deve ser reconciliado principalmente com:

- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-references-inventory.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/backoffice-admin-operational-runbooks.md`

Este documento complementa o canônico; não o substitui.

---

## Linha incremental do checkpoint

## 1. Consolidação do modelo admin session-first local

No início da frente, a Auth API ainda exigiu correções para estabilizar o fluxo admin local.

Problemas enfrentados:

- `ReferenceError: Cannot access 'dbConfig' before initialization`
- imports quebrados para `src/routes/config/auth.js`
- export inexistente `AUTH_DEV_ADMIN_BOOTSTRAP_ENABLED`
- falhas de bootstrap por indisponibilidade do MariaDB local
- drift de schema local em `sessions`
- problemas em `ops/stop.sh` e em comportamento de volumes Docker

Correções aplicadas:

- ajuste de imports e exports do contexto de auth
- revisão do fluxo `dev.bootstrap-session`
- endurecimento do `ops/dev.sh` para aguardar MariaDB ficar pronto
- limpeza de volumes para reconciliar shape local do banco
- validação local bem-sucedida de:
  - `POST /auth/dev/bootstrap-session`
  - `GET /auth/session`
  - cookie `hsc_admin_session`

Resultado:

- o modelo admin session-first ficou validado localmente
- o Backoffice passou a ter uma superfície real de sessão para consumir

---

## 2. Endurecimento antes do PR-2 e deploy inicial do backend

A Auth API e o Backoffice passaram por endurecimentos prévios de contrato, docs e operação.

Mudanças relevantes:

- revisão de documentação de contratos administrativos
- endurecimento de Auth API e Backoffice antes da publicação
- deploy inicial do backend com checagem de commits pendentes

Problema relevante em produção:

- primeiro deploy falhou porque o MariaDB no Lightsail não estava aceitando conexão em `127.0.0.1:3306`

Correção aplicada:

- restart do `mariadb`
- validação de socket/listen em `127.0.0.1:3306`
- restart do serviço `hsc-auth-api`
- revalidação de `/health`, `/content/news`, `/content/seasons`, `/content/seasons/active`

Resultado:

- o backend voltou a responder corretamente em produção
- `v0.4.0` consolidou a base do runtime dinâmico já publicado

---

## 3. Publicação do Backoffice em subdomínio próprio

Foi tomada a decisão de separar o Backoffice em subdomínio próprio na Hostinger, em vez de usar preset de deploy automático.

Decisões operacionais:

- uso de `Site PHP/HTML personalizado` na criação do subdomínio
- uso de `backoffice.haxixesmokeclub.com`
- upload manual do build Angular em `public_html`
- manutenção manual do `.htaccess`

Passos relevantes:

- limpeza da raiz do site
- publicação de página de prova
- criação de `.htaccess` com fallback SPA
- confirmação de que `/auth/callback` e `/dashboard` deixam de retornar 404 do servidor
- upload do build Angular correto a partir de `dist/backoffice-admin/browser`

Resultado:

- o Backoffice passou a operar como SPA publicada em subdomínio próprio
- a fronteira entre frontend administrativo e Auth API ficou formalizada também na infraestrutura

---

## 4. Correção de base URL e CORS para o Backoffice publicado

Após a publicação do Backoffice, o frontend ainda tentava chamar o próprio host do subdomínio em vez da Auth API.

Problema identificado:

- `POST /auth/magic-link/request` estava indo para `https://backoffice.haxixesmokeclub.com/...`
- a resposta era HTML da SPA, não JSON da API

Correção aplicada no Backoffice:

- introdução de `API_BASE_URL`
- uso explícito de `https://auth-api.haxixesmokeclub.com` em produção
- manutenção do fluxo local com base vazia e proxy/local runtime

Em seguida, surgiu o erro real de CORS:

- a Auth API não autorizava corretamente o origin do Backoffice publicado
- posteriormente também faltava `Access-Control-Allow-Credentials: true`

Correções aplicadas no backend:

- adoção de `ALLOWED_ORIGINS`
- inclusão de `https://backoffice.haxixesmokeclub.com` na allowlist de produção
- ajuste de `src/config/cors.js` para `credentials: true`

Release associada:

- **`v0.4.1`**

Resultado:

- o Backoffice publicado passou a chamar `POST /auth/magic-link/request` corretamente
- o preflight `OPTIONS` passou a retornar os headers esperados
- o request deixou de falhar por CORS

---

## 5. Implementação do fluxo de magic link admin

O fluxo real de magic link admin foi construído na Auth API com duas superfícies centrais:

- `POST /auth/magic-link/request`
- `GET /auth/magic-link/consume`

Componentes principais introduzidos:

- tabela `magic_links`
- camada `src/db/magicLinks.js`
- contrato `src/services/auth/magicLinkContract.js`
- delivery adapter `src/services/auth/magicLinkDelivery.js`
- consumo do link criando sessão administrativa

Fluxo funcional alvo:

1. operador informa email no Backoffice
2. backend responde com mensagem genérica de anti-enumeração
3. backend emite magic link one-time use
4. operador recebe email
5. operador clica no link
6. backend consome o token e cria sessão
7. backend redireciona para `/auth/callback?status=ok`
8. Backoffice resolve sessão e navega para `/dashboard`

Release associada:

- **`v0.4.1`**

Resultado:

- a primeira versão funcional do magic link admin foi colocada na linha de release

---

## 6. SMTP real via Hostinger

Após decisão explícita de usar a caixa Hostinger `no-reply@haxixesmokeclub.com`, o fluxo de email deixou de ser apenas logging local.

Etapas relevantes:

- criação da caixa `no-reply@haxixesmokeclub.com`
- coleta de parâmetros SMTP da Hostinger
  - `smtp.hostinger.com`
  - porta `465`
  - `secure: true`
- inclusão de envs SMTP em local e produção
- adoção de `nodemailer`
- adaptação de `src/services/auth/magicLinkDelivery.js` para envio real

Problema importante identificado:

- em produção, o backend continuava falhando com `smtp_host_missing`
- causa real: leitura de `process.env` cedo demais em `src/config/auth.js`

Correção aplicada:

- transformação das envs SMTP em getters de runtime (`getSmtpHost()`, `getSmtpPort()`, etc.)
- ajuste do delivery para ler a configuração em tempo de uso e não em tempo de import

Releases associadas:

- **`v0.4.2`** — envio real por SMTP
- **`v0.4.3`** — correção de runtime loading de env SMTP

Resultado:

- o email real passou a ser entregue via Hostinger SMTP
- o fluxo publicado ficou apto a enviar magic links de fato

---

## 7. Sessão cross-subdomain e callback do Backoffice

Mesmo com email entregue e consume bem-sucedido, o Backoffice ainda não conseguia resolver a sessão de forma consistente.

Problemas identificados:

- cookie emitido no `consume`, mas não reutilizado em `GET /auth/session`
- `SameSite=Lax` inadequado para o fluxo entre subdomínios em produção
- race condition curta no callback, onde o primeiro carregamento ainda não via a sessão pronta

Correções aplicadas:

### No backend

- revisão de `src/utils/sessionCookie.js`
- emissão de cookie com:
  - `Secure`
  - `SameSite=None`
  - `HttpOnly`
  - `Path=/`
  - `Max-Age` consistente
- fallback para `SameSite=Lax` em HTTP local

Release associada:

- **`v0.4.4`**

### No frontend

- `GET /auth/session` com `withCredentials: true`
- callback page com retry curto para `reloadSession()`
- navegação automática para `/dashboard` após autenticação bem-sucedida

Resultado:

- o fluxo publicado deixou de exigir `F5`
- o dashboard passou a abrir logado automaticamente após clique no magic link

---

## 8. Drift de schema em `sessions` e correção permanente

Durante o consume do magic link em produção, surgiu o erro:

- `Unknown column 'token_hash' in 'INSERT INTO'`

Diagnóstico:

- a tabela `sessions` em produção estava em shape legado
- `schema_meta.version` já havia avançado, então a reconciliação antiga em `v6` não rodava mais
- foi necessário patch manual inicial de `ALTER TABLE sessions ...`

Correção permanente implementada:

- criação da reconciliação **`v8`** em `src/db/schema.js`
- adição/garantia de:
  - `token_hash`
  - `revoked_at`
  - `updated_at`
  - índice único em `token_hash`

Release associada:

- **`v0.4.5`**

Resultado:

- o drift de `sessions` deixou de depender de patch manual
- ambientes legados passaram a ter caminho de reconciliação explícito

---

## 9. Decisão arquitetural: abandonar `schema.js` como mecanismo principal

Foi explicitamente decidido que manter crescimento indefinido de `src/db/schema.js` não era sustentável.

Motivação:

- arquivo crescendo indefinidamente
- mistura de runtime com evolução histórica do banco
- dificuldade de manutenção futura
- risco de drift entre `schema_meta` e shape real
- necessidade de um modelo único de migrations

Decisão tomada:

- adotar **SQL migrations por arquivo como fonte principal de evolução de schema**
- congelar `src/db/schema.js` como legado/compatibilidade
- mover o fluxo operacional para migrations explícitas no release/deploy

Essa decisão foi reforçada por:

- baseline do schema atual
- runner novo de migrations
- prova real em produção com a migration `0002`

---

## 10. Fundação do novo sistema de migrations

A primeira fundação concreta do novo modelo incluiu:

- diretório canônico `db/migrations/`
- runner `scripts/migrate.js`
- tabela `schema_migrations`
- script `db:migrate` no `package.json`
- `ops/deploy-auth.sh` executando `npm run db:migrate`
- `ops/release.sh` executando migrations locais antes do smoke
- `src/db/bootstrap.js` reduzido a readiness check

Migrations iniciais introduzidas:

- `0000_schema_migrations_bootstrap.sql`
- `0001_baseline_v0_4_5.sql`

Baseline em ambientes existentes:

- ambientes local e produção foram marcados em `schema_migrations`
- o ambiente local teve um resíduo antigo removido para ficar coerente com o novo modelo

Governança adicional:

- banner explícito no topo de `src/db/schema.js` marcando-o como legado
- criação de `docs/db-migrations-policy.md`

Release associada:

- **`v0.4.6`**

Resultado:

- o cutover operacional do modelo de banco foi concluído
- runtime deixou de ser responsável principal por evoluir schema

---

## 11. Primeira migration real pós-baseline

Para provar o novo fluxo no dia a dia, foi criada a primeira migration real do novo modelo:

- `0002_magic_links_add_used_at_index.sql`

Conteúdo:

- criação do índice `idx_magic_links_used_at` em `magic_links.used_at`

Validação local:

- migration aplicada com sucesso pelo runner
- índice visível em `SHOW INDEX FROM magic_links`

Promoção e release:

- merge em `develop`
- promoção para `main`
- release **`v0.4.7`**
- deploy no Lightsail executando `npm run db:migrate`

Validação em produção:

- `schema_migrations` passou a conter:
  - `0000_schema_migrations_bootstrap.sql`
  - `0001_baseline_v0_4_5.sql`
  - `0002_magic_links_add_used_at_index.sql`
- índice `idx_magic_links_used_at` confirmado em `magic_links`

Resultado:

- o novo modelo de migrations foi provado fim a fim em produção

---

## Releases do checkpoint

A sequência de releases associadas a este checkpoint foi:

- **`v0.4.0`** — base do backend dinâmico já publicada e revalidada após correção do MariaDB
- **`v0.4.1`** — magic link admin + CORS credentialed para o Backoffice
- **`v0.4.2`** — SMTP real via Hostinger
- **`v0.4.3`** — correção de runtime loading das envs SMTP
- **`v0.4.4`** — cookie seguro cross-subdomain para sessão admin
- **`v0.4.5`** — reconciliação permanente de `sessions` (`v8`)
- **`v0.4.6`** — fundação operacional das SQL migrations
- **`v0.4.7`** — primeira migration real pós-baseline (`0002`)

---

## Branches e PRs relevantes

As linhas de trabalho principais do checkpoint incluíram, entre outras:

- `feature/auth-magic-link-cors`
- `feature/auth-magic-link-smtp`
- `fix/smtp-env-runtime-loading`
- `fix/session-cookie-cross-subdomain`
- `fix/schema-sessions-v8-reconcile`
- `feat/sql-migrations-foundation`
- `docs/freeze-legacy-schema`
- `feat/migration-add-magic-links-used-at-index`

Também foram executadas branches de promoção/reconciliação para:

- reconciliar `develop` com `main`
- promover `develop` para `main` em releases específicas
- corrigir cenários em que fast-forward não era possível e foi necessário merge commit explícito

---

## Problemas importantes encontrados durante a frente

Os problemas mais relevantes do checkpoint foram:

### Backend / local

- imports quebrados em auth dev/local
- bootstrap falhando por banco ainda não pronto
- drift local de schema causado por volumes antigos

### Backend / produção

- MariaDB indisponível na primeira tentativa de deploy
- `smtp_host_missing` por leitura precoce de env
- `Unknown column 'token_hash' in 'INSERT INTO'` em `sessions`

### Frontend / Backoffice

- base URL apontando para o host errado do frontend
- CORS sem allowlist/cookies adequados
- callback com race condition exigindo `F5`

### Operação / branching

- promoções entre `develop` e `main` nem sempre puderam ser feitas por fast-forward
- foi necessário usar `--no-ff` quando o histórico divergiu

---

## Estado final validado ao fim do checkpoint

Ao fim deste checkpoint, o estado validado é:

### Auth API

- publicada e operacional em produção
- session-first admin funcional
- magic link request/consume funcional
- CORS publicado aceitando o Backoffice
- cookie cross-subdomain funcional
- SMTP Hostinger funcional
- `sessions` reconciliada permanentemente

### Backoffice Admin

- publicado em `backoffice.haxixesmokeclub.com`
- `/login` funcional
- `/auth/callback` funcional
- navegação para `/dashboard` funcional após consumo do magic link
- integração real com a Auth API em produção

### Banco / migrations

- baseline aplicado/marcado em ambientes existentes
- runner funcional em local e produção
- deploy usando `npm run db:migrate`
- primeira migration real aplicada em produção

### Governança

- `schema.js` explicitamente legado
- política de migrations documentada
- novo modelo operacional de schema ativo

---

## Limites e pendências após o checkpoint

Mesmo com o checkpoint grande concluído, ainda restam pontos naturais de continuidade:

- atualizar os documentos canônicos de `04-infra-aws-lightsail`
- atualizar os documentos canônicos de `05-backoffice-admin`
- decidir o destino final de `schema_meta` e do legado remanescente de `schema.js`
- continuar provando o novo modelo com migrations úteis de negócio
- eventualmente registrar ADR formal do cutover para SQL migrations, se desejado

Este impl-log não afirma que todo o legado foi removido. Ele registra que o **mecanismo principal** já mudou.

---

## Critério de pronto deste impl-log

Este impl-log está pronto quando:

- a trilha do checkpoint fica legível de ponta a ponta
- as releases associadas ficam explícitas
- os problemas reais e suas correções ficam rastreáveis
- os documentos canônicos a atualizar ficam claramente identificados
- o estado final fica descrito sem ambiguidade operacional relevante

---

## Última revisão

- Data: 2026-03-20
- Status: checkpoint consolidado
- Natureza: impl-log incremental de grande checkpoint
