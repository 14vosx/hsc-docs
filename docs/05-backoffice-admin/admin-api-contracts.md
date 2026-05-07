# Admin API Contracts

## Objetivo

Definir o contrato canônico entre o HSC Backoffice Admin e a HSC Auth API para superfícies administrativas publicadas.

Este documento cobre:

- autenticação administrativa baseada em sessão
- introspecção de sessão
- request de magic link
- superfícies administrativas de users já publicadas
- superfícies administrativas auxiliares já publicadas
- regras de transporte, cookies, CORS e erros esperados

Este documento não cobre:

- contratos públicos de conteúdo
- detalhes internos de implementação Express/Angular
- contratos futuros ainda não publicados

---

## Navegação

- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/backoffice-admin-frontend-structure.md`
- `docs/95-impl-log/2026-03-24-local-auth-users-management-checkpoint.md`

---

## Escopo

Este documento descreve o **contrato publicado** consumido pelo Backoffice Admin.

A leitura correta é:

- o backend é a autoridade final
- o frontend consome contratos HTTP explícitos
- autenticação administrativa é baseada em cookie de sessão
- mutações administrativas exigem sessão válida e papel aceito pelo backend
- no estado publicado atual, o shell administrativo continua admin-only

---

## Estado atual publicado

No estado publicado atual, a Auth API expõe para o Backoffice Admin:

### Auth / sessão
- `POST /auth/magic-link/request`
- `GET /auth/magic-link/consume`
- `GET /auth/session`

### Desenvolvimento local
- `POST /auth/dev/bootstrap-session`

### Admin schema / suporte
- `GET /admin/schema`

### Admin uploads
- `POST /admin/uploads`

### Admin users
- `GET /admin/users`
- `POST /admin/users`
- `PATCH /admin/users/:id`

Outras superfícies administrativas podem existir no backend, mas este documento prioriza o contrato efetivamente usado pela linha atual do Backoffice Admin publicado.

---

## Regras gerais de transporte

### Origem e credenciais

O Backoffice Admin publicado consome a Auth API com:

- `withCredentials: true`
- cookie HTTP-only de sessão
- CORS com allowlist explícita de origens administrativas

### Métodos efetivamente exigidos pela linha publicada

Para o fluxo administrativo atual, a política CORS precisa aceitar:

- `GET`
- `POST`
- `PATCH`
- `OPTIONS`

### Headers esperados

Headers mínimos aceitos na linha publicada:

- `Content-Type`
- `Authorization`

### Content-Type

Quando há body JSON, o contrato esperado é:

- `Content-Type: application/json`

---

## Cookie de sessão administrativa

A sessão administrativa é mantida por cookie emitido pela Auth API.

Características esperadas da linha publicada:

- cookie HTTP-only
- `Secure` em ambiente publicado
- `SameSite` compatível com o fluxo cross-subdomain
- reutilização do cookie nas chamadas subsequentes para `/auth/session` e superfícies `/admin/*`

O frontend **não** deve depender de leitura direta do cookie.
A fonte de verdade para “usuário autenticado ou não” é o resultado de `GET /auth/session`.

---

## Auth — request de magic link

### Endpoint

```http
POST /auth/magic-link/request
```

### Objetivo

Solicitar link administrativo de entrada por email.

### Request body

```json
{
  "email": "admin@example.com"
}
```

### Resposta esperada

A resposta é intencionalmente genérica para evitar enumeração de contas.

Exemplo:

```json
{
  "ok": true,
  "message": "If the account is allowed, a sign-in link has been sent."
}
```

### Regras importantes

* magic link **não cria usuário**
* magic link apenas autentica usuário **já existente e elegível**
* no estado publicado atual, elegível significa:

  * usuário existente
  * `role = "admin"`

### Observações

* reutilização do mesmo token não é suportada
* token já consumido ou expirado deve resultar em erro de callback/consume
* o fluxo canônico do frontend é:

  1. request do magic link
  2. clique no email
  3. callback
  4. introspecção de sessão em `/auth/session`

---

## Auth — consume de magic link

### Endpoint

```http
GET /auth/magic-link/consume?token=...
```

### Objetivo

Consumir token one-time de autenticação administrativa.

### Comportamento esperado

Em caso de sucesso:

* cria sessão administrativa
* emite cookie
* redireciona para o callback do Backoffice

Em caso de falha:

* redireciona com status/erro de link inválido ou expirado

### Observações

* token é one-time use
* consumir via terminal antes do navegador invalida o clique posterior no email
* o frontend não deve tentar inferir sucesso apenas pelo redirect; ele deve revalidar a sessão em `/auth/session`

---

---

## Admin uploads

### `POST /admin/uploads`

Contrato protegido da Auth API para upload administrativo de imagens.

Objetivo:

- receber uma imagem enviada pelo Backoffice Admin;
- validar tipo, tamanho e assinatura real do arquivo;
- salvar em storage local controlado pela Auth API;
- retornar uma URL pública pronta para persistência em entidades de domínio.

Uso previsto:

- `news.image_url`, já integrado visualmente no Backoffice News;
- `seasons.cover_image_url`, já integrado visualmente no Backoffice Seasons.

Ainda pendente:

- materialização runtime/prod do ETL atualizado, validação pública real da Static API v2 e validação visual pública do Portal com dados reais quando aplicável.

Este endpoint não altera `news`, `seasons` ou qualquer entidade funcional diretamente.

Request:

- `Content-Type: multipart/form-data`;
- campo `file`.

Tipos aceitos no primeiro corte:

- `image/jpeg`;
- `image/png`;
- `image/webp`.

SVG não é aceito neste corte.

Variáveis operacionais da Auth API:

- `UPLOAD_DIR`;
- `UPLOAD_PUBLIC_PATH`;
- `UPLOAD_PUBLIC_BASE_URL`;
- `UPLOAD_MAX_BYTES`.

Resposta de sucesso:

```json
{
  "ok": true,
  "url": "http://127.0.0.1:3000/uploads/20260507T171550813Z-21395de1a0ec0224.png",
  "path": "/uploads/20260507T171550813Z-21395de1a0ec0224.png",
  "filename": "20260507T171550813Z-21395de1a0ec0224.png",
  "size": 70,
  "mimetype": "image/png"
}
```

Erros esperados:

- `401 Unauthorized`: sem autenticação administrativa;
- `400 missing_file`: nenhum arquivo enviado no campo `file`;
- `400 invalid_file_type`: MIME declarado não permitido;
- `400 invalid_file_signature`: assinatura real do arquivo inválida;
- `400 file_type_mismatch`: MIME declarado e assinatura real divergem;
- `413 file_too_large`: arquivo maior que `UPLOAD_MAX_BYTES`;
- `500 audit_failed`: arquivo removido porque a auditoria administrativa falhou.

Invariantes de segurança:

- rota protegida por `requireAdmin`;
- exigência de `db_ready` antes da mutação;
- auditoria administrativa com `action = upload.create`;
- limite de tamanho via parser multipart;
- campo único esperado: `file`;
- MIME declarado restrito;
- validação por assinatura real do arquivo;
- nome final gerado pelo backend;
- nome original do usuário não é usado como path final;
- bloqueio de SVG;
- publicação estática com `X-Content-Type-Options: nosniff`.

Fronteira com Backoffice:

1. operador escolhe imagem no Backoffice;
2. Backoffice envia arquivo para `POST /admin/uploads`;
3. Auth API retorna `url`;
4. Backoffice persiste essa URL no campo funcional adequado.

Estado atual:

- News já usa esta fronteira para `image_url` no formulário administrativo;
- Seasons já usa esta fronteira para `cover_image_url` no formulário administrativo.

## Admin seasons — contrato de capa

### Superfícies

O contrato de Seasons já inclui `cover_image_url` nas seguintes superfícies administrativas:

- `GET /admin/seasons`
- `GET /admin/seasons/:slug`
- `POST /admin/seasons`
- `PATCH /admin/seasons/:slug`

### Leitura

`GET /admin/seasons` e `GET /admin/seasons/:slug` incluem `cover_image_url` na resposta.

Exemplo de item administrativo:

```json
{
  "id": 1,
  "slug": "season-smoke-local",
  "name": "Season Smoke Local",
  "description": "Season temporaria para smoke local.",
  "cover_image_url": null,
  "start_at": "2026-05-04T00:00:00.000Z",
  "end_at": "2026-05-31T23:59:59.000Z",
  "status": "draft",
  "created_at": "2026-05-04T12:00:00.000Z",
  "updated_at": "2026-05-04T12:00:00.000Z"
}
```

### Criação e edição

`POST /admin/seasons` aceita `cover_image_url` como campo opcional.

`PATCH /admin/seasons/:slug` aceita `cover_image_url` como campo editável.

Para limpar capa, enviar:

```json
{
  "cover_image_url": null
}
```

Regra importante:

- `cover_image_url: null` limpa a capa da Season.
- Seasons `closed` continuam read-only no Backoffice e não devem permitir upload ou limpeza pela UI.
- O código-fonte do ETL em `hsc-cs2-etl` já propaga `cover_image_url` nos JSONs estáticos de Seasons da Static API v2.
- O Portal Angular já está source-ready para consumir esse campo: `SeasonDto` tipa `cover_image_url`, `seasonCoverImage(...)` prioriza `cover_image_url`, cards/heróis de Seasons e Ranking usam `--season-cover`, e `npm run build` passou.
- Seguem pendentes a materialização runtime/prod do ETL atualizado, a validação pública real em `/api/cs2/v2/...` e a validação visual pública do Portal com dados reais.

## Auth — introspecção de sessão

### Endpoint

```http
GET /auth/session
```

### Objetivo

Retornar o estado administrativo atual da sessão.

### Sem sessão válida

Resposta típica:

```json
{
  "authenticated": false
}
```

Status esperado:

* `401` ou equivalente não autenticado

### Com sessão válida

Resposta típica:

```json
{
  "authenticated": true,
  "user": {
    "id": "2",
    "email": "admin@example.com",
    "name": "Admin Name"
  },
  "role": "admin"
}
```

### Contrato lógico consumido pelo frontend

Campos relevantes:

* `authenticated: boolean`
* `user.id`
* `user.email`
* `user.name`
* `role`

### Papéis reconhecidos

O modelo atual reconhece:

* `viewer`
* `editor`
* `admin`

### Observação importante

Embora o enum reconheça `viewer`, `editor` e `admin`, o estado efetivo publicado do shell administrativo continua admin-only.

---

## Auth — bootstrap dev local

### Endpoint

```http
POST /auth/dev/bootstrap-session
```

### Objetivo

Criar/promover sessão administrativa local para desenvolvimento.

### Uso correto

* somente em desenvolvimento local
* não faz parte do fluxo administrativo publicado para produção
* serve para acelerar smoke local do Backoffice

### Comportamento esperado

* cria ou promove o usuário local configurado para admin
* emite cookie de sessão
* permite navegar localmente no shell administrativo

---

## Admin schema

### Endpoint

```http
GET /admin/schema
```

### Objetivo

Superfície administrativa auxiliar para diagnóstico/inspeção de schema.

### Regras

* protegida por sessão administrativa válida
* útil para smoke administrativo pós-deploy
* não substitui documentação canônica de schema

---

## Admin users — listagem

### Endpoint

```http
GET /admin/users
```

### Objetivo

Listar usuários administrativos e relacionados ao Backoffice.

### Resposta esperada

```json
{
  "ok": true,
  "count": 2,
  "items": [
    {
      "id": 5,
      "email": "user@example.com",
      "display_name": "User Name",
      "role": "viewer",
      "created_at": "2026-03-24T18:00:00.000Z",
      "updated_at": "2026-03-24T18:00:00.000Z"
    }
  ]
}
```

### Campos relevantes por item

* `id`
* `email`
* `display_name`
* `role`
* `created_at`
* `updated_at`

### Observações

* a linha publicada atual não expõe paginação nesta superfície
* o frontend publicado trata essa resposta como lista integral para o primeiro corte da área `/users`

---

## Admin users — criação

### Endpoint

```http
POST /admin/users
```

### Objetivo

Criar usuário administrativo/domínio para a linha atual do Backoffice.

### Request body

```json
{
  "email": "user@example.com",
  "display_name": "User Name",
  "role": "viewer"
}
```

### Regras de validação esperadas

* `email` obrigatório e válido
* `display_name` obrigatório
* `role` aceito dentro de:

  * `viewer`
  * `editor`
  * `admin`

### Resposta de sucesso

```json
{
  "ok": true,
  "item": {
    "id": 10,
    "email": "user@example.com",
    "display_name": "User Name",
    "role": "viewer",
    "created_at": "2026-03-24T18:00:00.000Z",
    "updated_at": "2026-03-24T18:00:00.000Z"
  }
}
```

### Respostas de erro esperadas

#### campos ausentes

```json
{
  "ok": false,
  "error": "missing_fields"
}
```

#### email inválido

```json
{
  "ok": false,
  "error": "invalid_email"
}
```

#### nome inválido

```json
{
  "ok": false,
  "error": "invalid_display_name"
}
```

#### papel inválido

```json
{
  "ok": false,
  "error": "invalid_role"
}
```

#### email já existente

```json
{
  "ok": false,
  "error": "email_already_exists"
}
```

#### erro genérico de banco

```json
{
  "ok": false,
  "error": "db_error"
}
```

### Observação funcional importante

Criar usuário com `role = "viewer"` ou `role = "editor"` **não** implica acesso administrativo ao Backoffice no estado publicado atual.

---

## Admin users — atualização parcial

### Endpoint

```http
PATCH /admin/users/:id
```

### Objetivo

Atualizar parcialmente um usuário existente.

### Campos aceitos no body

* `email`
* `display_name`
* `role`

### Exemplo — alterar role

```json
{
  "role": "editor"
}
```

### Exemplo — renomear

```json
{
  "display_name": "Novo Nome"
}
```

### Exemplo — alterar email

```json
{
  "email": "novo-email@example.com"
}
```

### Resposta de sucesso

```json
{
  "ok": true,
  "item": {
    "id": 10,
    "email": "novo-email@example.com",
    "display_name": "Novo Nome",
    "role": "editor",
    "created_at": "2026-03-24T18:00:00.000Z",
    "updated_at": "2026-03-24T18:10:00.000Z"
  }
}
```

### Respostas de erro esperadas

#### id inválido

```json
{
  "ok": false,
  "error": "invalid_id"
}
```

#### nenhum campo enviado

```json
{
  "ok": false,
  "error": "no_fields_to_update"
}
```

#### email inválido

```json
{
  "ok": false,
  "error": "invalid_email"
}
```

#### nome inválido

```json
{
  "ok": false,
  "error": "invalid_display_name"
}
```

#### role inválida

```json
{
  "ok": false,
  "error": "invalid_role"
}
```

#### registro não encontrado

```json
{
  "ok": false,
  "error": "not_found"
}
```

#### email duplicado

```json
{
  "ok": false,
  "error": "email_already_exists"
}
```

#### erro genérico de banco

```json
{
  "ok": false,
  "error": "db_error"
}
```

---

## Regras de autorização do contrato

### Estado efetivo publicado

No estado publicado atual:

* `GET /admin/users` exige sessão administrativa aceita pelo backend
* `POST /admin/users` exige sessão administrativa aceita pelo backend
* `PATCH /admin/users/:id` exige sessão administrativa aceita pelo backend

### Interpretação correta

Embora o enum de `role` já reconheça `viewer`, `editor` e `admin`, a linha publicada continua com autorização efetiva admin-only para shell administrativo e mutações administrativas.

---

## Regras de compatibilidade com legado

A linha publicada atual já passou pelas seguintes reconciliações:

### `0003_admin_audit_log.sql`

* criação/alinhamento da tabela de audit administrativo

### `0004_users_role_enum_reconcile.sql`

* reconciliação do legado de `users.role`
* conversão de `user` para `viewer`
* enum final publicado:

  * `viewer`
  * `editor`
  * `admin`

### Hotfix de CORS

* liberação de `PATCH` no CORS da Auth API
* necessária para:

  * alterar role
  * renomear
  * alterar email pela UI publicada

---

## Relação com o frontend publicado

A área `/users` do Backoffice publicado depende explicitamente deste contrato:

* `GET /admin/users`
* `POST /admin/users`
* `PATCH /admin/users/:id`

Capacidades publicadas na UI:

* listagem real de usuários
* criação
* alteração de role
* renomeação
* alteração de email

O frontend deve tratar a Auth API como autoridade final para:

* elegibilidade de login
* mutações administrativas
* validação de papel
* CORS e aceitação real de método

---

## Critério de pronto

Este documento está correto quando:

* reflete apenas contratos administrativos realmente publicados
* deixa claro que magic link não cria usuário
* deixa claro que login administrativo publicado continua admin-only
* descreve corretamente o CRUD básico de users
* registra reconciliações de schema relevantes para a linha publicada
* não mistura contrato administrativo com wish-list futura

---

## Última revisão

* 2026-03-24
* reconciliado após rollout completo de admin users management e hotfixes `v0.4.8`, `v0.4.9` e `v0.4.10`
