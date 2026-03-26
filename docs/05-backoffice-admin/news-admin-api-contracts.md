# News Admin API Contracts

## Objetivo

Definir o contrato canônico do domínio administrativo de `news` no ecossistema HSC.

Este documento existe para:

- registrar as superfícies administrativas reais de `news` já reconciliadas no runtime
- separar claramente o contrato admin do contrato público de conteúdo
- documentar payloads, respostas, invariantes e erros observados
- explicar como o Backoffice Admin deve consumir o domínio
- registrar a fronteira entre Auth API, Backoffice Admin e Portal público
- orientar a expansão futura do domínio sem reabrir ambiguidade contratual

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Backoffice Admin](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Documentos diretamente relacionados
- [Admin API Contracts](./admin-api-contracts.md)
- [News Admin Integration and Evolution](./news-admin-integration-and-evolution.md)
- [Auth, RBAC and Guards](./auth-rbac-and-guards.md)
- [Backoffice Admin Frontend Structure](./backoffice-admin-frontend-structure.md)
- [Backoffice Admin References Inventory](./backoffice-admin-references-inventory.md)

### Relações com outros contextos
- [Auth API Operations](../04-infra-aws-lightsail/auth-api-operations.md)
- [Nginx Publishing and Cache](../03-portal-estatico/nginx-publishing-cache.md)
- [ETL Bash Pipeline](../03-portal-estatico/etl-bash-pipeline.md)
- [Portal Estático Operational Runbooks](../03-portal-estatico/portal-estatico-operational-runbooks.md)

---

## Escopo

Este documento cobre:

- contratos administrativos reais de `news`
- contratos públicos canônicos de `news` na Auth API
- shape de requests e responses reconciliados
- lifecycle `draft` ↔ `published`
- comportamento observado de autenticação, validação e transição de estado
- forma correta de consumo pelo Backoffice Admin
- limites e lacunas ainda abertas do domínio

Este documento não cobre em profundidade:

- implementação interna Express ou SQL
- UI detalhada do formulário Angular
- operação detalhada do mirror same-origin do Portal
- fluxos futuros ainda não publicados
- backlog do domínio

Esses tópicos vivem em documentos adjacentes.

---

## Leitura correta do domínio

O domínio `news` possui **três superfícies diferentes** e elas não devem ser misturadas.

### 1. Superfície administrativa autenticada

Consumida pelo Backoffice Admin na Auth API:

- `GET /admin/news`
- `POST /admin/news`
- `PATCH /admin/news/:id`
- `POST /admin/news/:id/publish`
- `POST /admin/news/:id/unpublish`
- `DELETE /admin/news/:id`

### 2. Superfície pública canônica de conteúdo

Exposta pela Auth API como fonte dinâmica canônica:

- `GET /content/news`
- `GET /content/news/:slug`

### 3. Superfície pública same-origin do Portal

Servida pelo Nginx do Portal a partir do cache público local:

- `GET /content/news/`
- `GET /content/news/<slug>/`

Regra importante:

- o **Backoffice Admin** consome a superfície `/admin/news*`
- a **fonte canônica dinâmica** de conteúdo continua sendo `/content/news*` na Auth API
- o **Portal público** consome o mirror same-origin `/content/news/` servido localmente
- o Backoffice **não** deve usar o mirror do Portal como fonte de gestão administrativa

---

## Estado atual reconciliado

No estado reconciliado atual, o domínio `news` já está funcional ponta a ponta para o ciclo administrativo básico.

Capacidades comprovadas:

- listagem administrativa protegida por sessão
- criação de draft
- edição parcial
- publicação
- despublicação
- remoção
- exposição pública canônica após publish
- remoção da superfície pública após unpublish
- detalhe público por `slug`
- retorno a lista vazia após delete

Leitura correta do checkpoint:

- `news` já possui contrato suficiente para habilitar a feature no Backoffice Admin
- o domínio já não está em fase puramente especulativa
- o contrato mínimo real já pode governar implementação frontend com segurança

---

## Regras gerais do contrato

### Transporte

O consumo administrativo deve seguir as regras transversais já estabelecidas em `admin-api-contracts.md`:

- `withCredentials: true`
- cookie de sessão HTTP-only
- CORS controlado pela Auth API
- `Content-Type: application/json` quando houver body

### Autenticação

As superfícies `/admin/news*` exigem sessão administrativa válida.

Sem sessão válida, o comportamento observado foi:

```json
{
  "ok": false,
  "error": "Unauthorized"
}
```

Status observado:

- `401 Unauthorized`

### Papel do backend

O backend permanece autoridade final para:

- autenticação
- autorização
- invariantes de transição
- shape final do recurso
- publicação e despublicação
- remoção
- auditoria transacional das mutações sensíveis

---

## Modelo lógico mínimo do recurso

Os campos efetivamente observados no domínio administrativo de `news` foram:

- `id`
- `slug`
- `title`
- `excerpt`
- `image_url`
- `status`
- `published_at`
- `created_at`
- `updated_at`

Na superfície pública canônica, os campos observados foram:

### Lista pública

- `slug`
- `title`
- `excerpt`
- `image_url`
- `published_at`

### Detalhe público

- `slug`
- `title`
- `excerpt`
- `content`
- `image_url`
- `published_at`

Leitura importante:

- `content` aparece no detalhe público
- `status` e `id` **não** aparecem no contrato público
- a superfície pública expõe apenas o que é relevante para consumo de conteúdo

---

## Invariantes canônicos do domínio

Os seguintes invariantes permanecem canônicos para `news`:

- `slug` deve ser único
- o fluxo administrativo parte de `draft`
- `publish` promove o item para `published`
- `unpublish` retorna o item para `draft`
- `published_at` deve refletir a publicação em UTC
- a listagem pública expõe apenas itens publicados
- a listagem administrativa pode conter drafts e publicados
- o Portal público consome o mirror same-origin, não a Auth API diretamente no browser

---

## Matriz de superfícies reconciliadas

| Método | Endpoint | Auth | Papel funcional |
|---|---|---:|---|
| GET | `/admin/news` | sim | listar itens administrativos |
| POST | `/admin/news` | sim | criar draft |
| PATCH | `/admin/news/:id` | sim | editar parcialmente |
| POST | `/admin/news/:id/publish` | sim | publicar item |
| POST | `/admin/news/:id/unpublish` | sim | retirar da superfície pública |
| DELETE | `/admin/news/:id` | sim | remover item |
| GET | `/content/news` | não | lista pública canônica |
| GET | `/content/news/:slug` | não | detalhe público canônico |
| GET | `/content/news/` | não | mirror same-origin do Portal |
| GET | `/content/news/<slug>/` | não | detalhe same-origin do Portal |

---

## Contrato administrativo — listagem

### Endpoint

```http
GET /admin/news
```

### Objetivo

Listar o estado administrativo atual do domínio `news`.

### Sem sessão válida

Resposta observada:

```json
{
  "ok": false,
  "error": "Unauthorized"
}
```

Status observado:

- `401 Unauthorized`

### Com sessão válida e sem itens

Resposta observada:

```json
{
  "ok": true,
  "count": 0,
  "items": []
}
```

### Campos esperados por item

Com base nas respostas reconciliadas, cada item administrativo tende a expor:

```json
{
  "id": 1,
  "slug": "news-crud-smoke-20260325",
  "title": "News CRUD Smoke 2026-03-25 Atualizada",
  "excerpt": null,
  "image_url": null,
  "status": "draft",
  "published_at": null,
  "created_at": "2026-03-25T19:11:24.000Z",
  "updated_at": "2026-03-25T19:12:50.000Z"
}
```

### Observações

- a resposta observada não expôs paginação
- a resposta observada usa `count` + `items`
- a listagem administrativa é a principal superfície de leitura canônica confirmada para o Backoffice no checkpoint atual

---

## Contrato administrativo — criação

### Endpoint

```http
POST /admin/news
```

### Objetivo

Criar um novo item de `news` em estado inicial de draft.

### Body mínimo confirmado

A validação observada no runtime confirmou que os campos mínimos obrigatórios são:

- `slug`
- `title`
- `content`

Exemplo mínimo válido:

```json
{
  "slug": "news-crud-smoke-20260325",
  "title": "News CRUD Smoke 2026-03-25",
  "content": "Primeiro draft de teste do fluxo news."
}
```

### Validação observada para body vazio

Request:

```json
{}
```

Resposta observada:

```json
{
  "ok": false,
  "error": "missing_fields",
  "required": ["slug", "title", "content"]
}
```

Status observado:

- `400 Bad Request`

### Resposta de sucesso observada

```json
{
  "ok": true,
  "id": 1,
  "slug": "news-crud-smoke-20260325",
  "status": "draft"
}
```

Status observado:

- `201 Created`

### Leitura correta

- a criação mínima confirmada ainda não exige `excerpt`
- a criação mínima confirmada ainda não exige `image_url`
- o backend já devolve `status: "draft"` imediatamente após a criação
- o frontend deve tratar essa resposta como sucesso suficiente para redirecionar, revalidar lista ou abrir edição

---

## Contrato administrativo — atualização parcial

### Endpoint

```http
PATCH /admin/news/:id
```

### Objetivo

Atualizar parcialmente um item de `news` já existente.

### Campos confirmados como mutáveis

Os seguintes campos foram confirmados no runtime:

- `title`
- `content`

Exemplo observado:

```json
{
  "title": "News CRUD Smoke 2026-03-25 Atualizada",
  "content": "Draft atualizado no teste completo de CRUD do recurso news."
}
```

### Resposta de sucesso observada

```json
{
  "ok": true,
  "item": {
    "id": 1,
    "slug": "news-crud-smoke-20260325",
    "title": "News CRUD Smoke 2026-03-25 Atualizada",
    "excerpt": null,
    "image_url": null,
    "status": "draft",
    "published_at": null,
    "created_at": "2026-03-25T19:11:24.000Z",
    "updated_at": "2026-03-25T19:12:50.000Z"
  }
}
```

Status observado:

- `200 OK`

### Observações importantes

- o recurso permaneceu em `draft` após edição
- `published_at` permaneceu `null`
- o retorno da mutação já contém shape administrativo suficiente para atualizar tabela, cache local ou estado do formulário

### Campos ainda não confirmados nesta revisão

Ainda não houve validação runtime explícita, neste checkpoint, para update de:

- `slug`
- `excerpt`
- `image_url`

Leitura canônica atual:

- não assumir esses campos como mutáveis até reconciliação explícita adicional

---

## Contrato administrativo — publish

### Endpoint

```http
POST /admin/news/:id/publish
```

### Objetivo

Publicar um item de `news` e torná-lo elegível para a superfície pública canônica.

### Resposta de sucesso observada

```json
{
  "ok": true,
  "item": {
    "id": 1,
    "slug": "news-crud-smoke-20260325",
    "title": "News CRUD Smoke 2026-03-25 Atualizada",
    "excerpt": null,
    "image_url": null,
    "status": "published",
    "published_at": "2026-03-25T19:14:35.000Z",
    "created_at": "2026-03-25T19:11:24.000Z",
    "updated_at": "2026-03-25T19:14:35.000Z"
  }
}
```

Status observado:

- `200 OK`

### Invariantes observados

- `status` passou de `draft` para `published`
- `published_at` foi preenchido
- `updated_at` foi alterado junto da publicação
- o item passou a aparecer em `GET /content/news`

---

## Contrato administrativo — unpublish

### Endpoint

```http
POST /admin/news/:id/unpublish
```

### Objetivo

Retirar um item publicado da superfície pública, retornando-o ao estado de draft.

### Resposta de sucesso observada

```json
{
  "ok": true,
  "item": {
    "id": 1,
    "slug": "news-crud-smoke-20260325",
    "title": "News CRUD Smoke 2026-03-25 Atualizada",
    "excerpt": null,
    "image_url": null,
    "status": "draft",
    "published_at": null,
    "created_at": "2026-03-25T19:11:24.000Z",
    "updated_at": "2026-03-25T19:18:32.000Z"
  }
}
```

Status observado:

- `200 OK`

### Invariantes observados

- `status` voltou para `draft`
- `published_at` voltou para `null`
- o item deixou de aparecer em `GET /content/news`

---

## Contrato administrativo — delete

### Endpoint

```http
DELETE /admin/news/:id
```

### Objetivo

Remover definitivamente um item de `news` da superfície administrativa.

### Resposta de sucesso observada

```json
{
  "ok": true,
  "deleted": 1
}
```

Status observado:

- `200 OK`

### Observações

- após o delete, `GET /admin/news` voltou a responder com `count: 0`
- a remoção foi efetiva tanto no estado administrativo quanto no ciclo público já despublicado

---

## Contrato público canônico — lista

### Endpoint

```http
GET /content/news
```

### Objetivo

Expor a lista pública canônica de notícias publicadas na Auth API.

### Sem itens publicados

Resposta observada:

```json
{
  "ok": true,
  "count": 0,
  "items": []
}
```

### Com item publicado

Resposta observada:

```json
{
  "ok": true,
  "count": 1,
  "items": [
    {
      "slug": "news-crud-smoke-20260325",
      "title": "News CRUD Smoke 2026-03-25 Atualizada",
      "excerpt": null,
      "image_url": null,
      "published_at": "2026-03-25T19:14:35.000Z"
    }
  ]
}
```

### Observações

- esta superfície expõe apenas itens publicados
- `id`, `status` e `content` não aparecem na lista pública
- `count` + `items` permanecem o shape reconciliado

---

## Contrato público canônico — detalhe

### Endpoint

```http
GET /content/news/:slug
```

### Objetivo

Expor o detalhe público canônico de uma notícia publicada.

### Resposta de sucesso observada

```json
{
  "ok": true,
  "item": {
    "slug": "news-crud-smoke-20260325",
    "title": "News CRUD Smoke 2026-03-25 Atualizada",
    "excerpt": null,
    "content": "Draft atualizado no teste completo de CRUD do recurso news.",
    "image_url": null,
    "published_at": "2026-03-25T19:14:35.000Z"
  }
}
```

### Observações

- o detalhe público expõe `content`
- o contrato público continua sem `id` e sem `status`
- esta é a fonte dinâmica canônica do conteúdo público antes do mirror same-origin do Portal

---

## Relação com o mirror same-origin do Portal

O Portal público não deve depender da Auth API diretamente no browser para consumir `news`.

A leitura correta continua sendo:

- a Auth API publica a fonte canônica dinâmica em `/content/news` e `/content/news/:slug`
- o ETL/espelhamento gera cache local
- o Nginx do Portal serve o conteúdo em:
  - `/content/news/`
  - `/content/news/<slug>/`

Isto protege:

- same-origin no browser
- desacoplamento de CORS no Portal público
- tolerância a indisponibilidade momentânea da Auth API, limitada pela staleness do cache

Regra importante:

- isso **não** muda o fato de que a fonte canônica do conteúdo continua sendo a Auth API
- o Backoffice Admin não deve trocar seu consumo administrativo pela superfície de mirror

---

## Uso correto pelo Backoffice Admin

### Tela `/news`

A tela de listagem deve consumir:

```http
GET /admin/news
```

Ela deve usar a resposta para:

- renderizar tabela
- mostrar status
- habilitar ações de publish/unpublish/delete
- navegar para criação e edição

### Tela `/news/new`

A criação deve consumir:

```http
POST /admin/news
```

Com body mínimo confirmado:

- `slug`
- `title`
- `content`

Após sucesso, o frontend pode:

- revalidar `GET /admin/news`
- navegar para edição
- mostrar feedback de criação concluída

### Tela `/news/:id/edit`

O fluxo de edição deve consumir:

```http
PATCH /admin/news/:id
```

Leitura importante do checkpoint atual:

- não foi reconciliado, nesta revisão, um `GET /admin/news/:id`
- portanto, a tela de edição não deve presumir esse endpoint como canônico até confirmação futura

Estratégias aceitáveis no checkpoint atual:

- hidratar a edição a partir da listagem já carregada, quando suficiente
- reter estado do item recém-criado/atualizado no fluxo do frontend
- adicionar leitura dedicada apenas quando essa superfície estiver publicada e reconciliada

### Ações de ciclo de vida

O frontend deve tratar publish e unpublish como mutações explícitas, não como simples patch de `status`.

Endpoints corretos:

- `POST /admin/news/:id/publish`
- `POST /admin/news/:id/unpublish`

Leitura correta:

- o lifecycle é governado por endpoints dedicados
- o frontend não deve tentar sintetizar publish/unpublish por conta própria

---

## Erros e respostas observadas

### Falta de autenticação

```json
{
  "ok": false,
  "error": "Unauthorized"
}
```

Status observado:

- `401`

### Falta de campos mínimos na criação

```json
{
  "ok": false,
  "error": "missing_fields",
  "required": ["slug", "title", "content"]
}
```

Status observado:

- `400`

### Lacuna explícita

Ainda não há, nesta revisão, reconciliação suficiente para promover a catálogo canônico completo de:

- erros de slug duplicado
- erros de `id` inexistente
- erros de publish inválido por estado
- erros de unpublish inválido por estado
- shape de erro de validação de `excerpt` e `image_url`

Regra importante:

- não inventar esses contratos
- expandir este documento quando o runtime real for validado

---

## Limites e lacunas abertas

Os seguintes pontos continuam abertos ou parcialmente confirmados:

### 1. Leitura admin por item

Ainda não foi confirmada uma superfície canônica de:

```http
GET /admin/news/:id
```

### 2. Campos opcionais mutáveis

Ainda não foi validado no runtime, neste checkpoint, o comportamento exato de:

- `excerpt`
- `image_url`
- alteração de `slug`

### 3. Paginação, ordenação e filtros

Ainda não houve confirmação de:

- paginação
- filtros
- ordenação explícita
- busca textual server-side

### 4. Catálogo completo de erros

Apenas parte do contrato de erro foi comprovada.

---

## Leitura canônica atual

Neste checkpoint, a leitura canônica segura é:

- `news` já possui CRUD administrativo básico utilizável
- o contrato mínimo de criação é `slug`, `title`, `content`
- o ciclo `draft` → `published` → `draft` está funcional
- `GET /content/news` e `GET /content/news/:slug` já refletem corretamente o estado publicado
- o domínio já é suficiente para a primeira entrega do Backoffice Admin
- expansões adicionais devem respeitar o contrato já reconciliado, não substituí-lo silenciosamente

---

## Critério de pronto

Este documento está saudável quando:

- separa claramente admin, conteúdo canônico e mirror same-origin
- registra apenas contratos realmente reconciliados
- documenta o corpo mínimo de criação e o lifecycle real
- deixa explícitas as lacunas restantes
- permite implementação frontend sem depender de inferência informal
- evita misturar wish-list futura com contrato já publicado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: `05-backoffice-admin`
- Domínio: `news`
- Última revisão: 2026-03-25
