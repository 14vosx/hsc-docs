# 2026-03-24 — Local Auth + Users Management Checkpoint

## Objetivo

Registrar, de forma estável e auditável, o checkpoint local desta frente de trabalho, cobrindo:

- entendimento e validação do fluxo administrativo por magic link
- confirmação explícita de que magic link não cria usuário
- implementação local do CRUD básico administrativo de usuários na Auth API
- implementação local da área `/users` no Backoffice Admin
- validação ponta a ponta local entre Backoffice e Auth API
- correção de drift de schema relacionada ao audit log administrativo

Este impl-log existe para preservar a trilha incremental desta entrega sem substituir os documentos canônicos de `04-infra-aws-lightsail` e `05-backoffice-admin`.

---

## Escopo

Este documento cobre:

- o entendimento consolidado do modelo atual de magic link admin
- as mudanças locais feitas na Auth API
- as mudanças locais feitas no Backoffice Admin
- os smokes locais executados
- o drift encontrado no banco local
- a correção aplicada por migration
- os commits locais gerados ao final do checkpoint

Este documento não cobre em profundidade:

- deploy em ambiente publicado
- promoção para `develop` ou `main`
- publicação de remote para o Backoffice
- refinamento visual da tela `/users`
- expansão futura de RBAC por domínio/ação

---

## Estado inicial do checkpoint

No início desta frente, havia duas necessidades principais:

1. esclarecer como o magic link realmente funciona no fluxo administrativo
2. abrir a gestão de usuários no Backoffice sem quebrar o modelo atual de autenticação

A dúvida central era se um novo usuário admin nascia automaticamente via magic link.

A conclusão consolidada neste checkpoint foi:

- o magic link **não cria usuário**
- ele apenas autentica um usuário **já existente e elegível**
- no estado atual do sistema, elegível para acesso administrativo significa estar com `role = 'admin'`

---

## Source of truth / evidências

As evidências diretas deste impl-log são:

- documentação canônica do repositório `hsc-docs`
- trilha de implementação local validada nesta frente
- smokes locais feitos via `curl`, navegador e Angular dev server
- branches locais abertas especificamente para esta entrega
- commits locais gerados ao final do checkpoint

Repositórios e branches envolvidos:

- `hsc-auth-api`
  - branch: `feat/auth-admin-users-crud`
- `hsc-backoffice-admin`
  - branch: `feat/backoffice-admin-users`

Commits locais gerados:

- `hsc-auth-api`
  - `d53486b` — `feat(users): add admin users management endpoints`
- `hsc-backoffice-admin`
  - `cb7bab3` — `feat(users): add backoffice admin users management page`

---

## Relações com outros documentos

Este impl-log deve ser reconciliado principalmente com:

- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/backoffice-admin-frontend-structure.md`
- `docs/05-backoffice-admin/backoffice-admin-operational-runbooks.md`

Este documento complementa o canônico; não o substitui.

---

## Entendimento consolidado do magic link

O fluxo administrativo validado neste checkpoint ficou assim:

1. operador informa email no Backoffice
2. Backoffice chama `POST /auth/magic-link/request`
3. Auth API responde de forma genérica
4. se o usuário existir e for elegível, o backend emite o magic link
5. o operador consome o link
6. a Auth API cria a sessão administrativa e emite o cookie
7. o Backoffice chama `GET /auth/session`
8. a UI só considera o operador autenticado após introspecção positiva da sessão

Conclusões práticas validadas:

- o token do magic link é one-time use
- reutilização do mesmo link gera erro de link inválido/expirado
- o fluxo real no navegador funciona quando o link é consumido uma única vez
- localmente, para desenvolvimento, o fluxo correto continua sendo `POST /auth/dev/bootstrap-session`

---

## Mudanças locais na Auth API

### Superfícies novas

Foram implementadas localmente as rotas administrativas:

- `GET /admin/users`
- `POST /admin/users`
- `PATCH /admin/users/:id`

### Wiring

O arquivo `src/routes/register.js` foi atualizado para registrar:

- `registerAdminUsersListRoute`
- `registerAdminUsersCreateRoute`
- `registerAdminUsersUpdateRoute`

### Regra de negócio preservada

O sistema permaneceu coerente com o modelo atual:

- usuário pode existir com `viewer`, `editor` ou `admin`
- porém, no estado atual, apenas `admin` é elegível para login administrativo via magic link
- as rotas admin também continuam protegidas por `requireAdmin(...)`

---

## Drift local encontrado e correção

Durante o smoke local do `POST /admin/users`, foi encontrado um `500 db_error`.

A causa real foi drift entre código e schema:

- o código já escrevia audit em `admin_audit_log`
- o banco local não possuía essa tabela
- o baseline aplicado localmente ainda não a criava

A correção aplicada foi a migration:

- `db/migrations/0003_admin_audit_log.sql`

Essa migration foi aplicada localmente com o env correto:

- `ENV_FILE=.env.local npm run db:migrate`

Resultado:

- tabela `admin_audit_log` criada com sucesso
- migration registrada em `schema_migrations`
- `POST /admin/users` passou a funcionar normalmente

---

## Mudanças locais no Backoffice Admin

### Navegação e rota

Foram adicionados:

- rota protegida `/users`
- item `Users` no sidebar administrativo

### Feature criada

Foi criada a feature:

- `src/app/features/users/`

Estrutura implementada:

- `data-access/users-admin-api.service.ts`
- `pages/users-page/users-page.component.ts`
- `pages/users-page/users-page.component.html`
- `pages/users-page/users-page.component.scss`

### Capacidades entregues na tela `/users`

No primeiro corte local, a tela passou a permitir:

- listagem real de usuários
- criação de usuário
- renomear usuário
- alterar email
- alterar role

A integração foi feita contra a Auth API local usando sessão/cookie com credenciais.

---

## Validações locais executadas

### Auth API

Validações executadas localmente:

- bootstrap dev de sessão admin
- listagem de usuários via `GET /admin/users`
- criação de usuário via `POST /admin/users`
- atualização de usuário via `PATCH /admin/users/:id`
- validação da migration `0003_admin_audit_log.sql`

### Backoffice

Validações executadas localmente:

- login local via `POST /auth/dev/bootstrap-session`
- acesso à rota protegida `/users`
- renderização da lista real de usuários
- criação de usuário pela UI
- alteração de role pela UI
- renomeação pela UI
- alteração de email pela UI
- build Angular bem-sucedido com `npm run build`

---

## Estado final ao fim do checkpoint

Ao final desta frente, o estado local validado ficou assim:

### Auth API local
- CRUD básico administrativo de usuários funcional
- audit administrativo funcional
- schema alinhado com a nova migration
- branch local pronta para promoção futura

### Backoffice local
- área `/users` funcional no primeiro corte
- integração ponta a ponta com a Auth API local validada
- fluxo de operação manual suficiente para gestão básica de usuários

### Regra funcional importante
No estado atual do sistema:

- `viewer` e `editor` podem existir e ser geridos
- mas não possuem elegibilidade para login administrativo por magic link
- apenas `admin` entra no Backoffice no modelo atual

---

## Pendências e reconciliações futuras

Após este impl-log, os próximos ajustes documentais mais prováveis são:

- explicitar `admin/users` em `docs/05-backoffice-admin/admin-api-contracts.md`
- reconciliar a área `/users` em `docs/05-backoffice-admin/backoffice-admin-frontend-structure.md`
- documentar melhor o bootstrap local e o uso de `ENV_FILE=.env.local` em `docs/04-infra-aws-lightsail/mariadb-local.md` e/ou runbooks adjacentes
- decidir se `editor` continuará sem acesso ao Backoffice ou se haverá expansão futura do RBAC administrativo

---

## Resumo executivo

Este checkpoint consolidou dois resultados importantes:

1. o entendimento correto do magic link administrativo
   - ele autentica sessão
   - não provisiona usuário
   - exige usuário pré-existente e elegível

2. a primeira entrega local de gestão de usuários
   - backend local pronto
   - frontend local pronto
   - fluxo ponta a ponta validado
   - drift de schema corrigido por migration

Com isso, a base local para evolução futura da gestão administrativa de usuários passou a existir de forma funcional, auditável e alinhada ao padrão atual do projeto.
