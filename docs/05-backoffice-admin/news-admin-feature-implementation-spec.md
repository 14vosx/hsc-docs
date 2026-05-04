# News Admin Feature Implementation Spec

## Objetivo

Documentar a especificação canônica de implementação da feature `news` no contexto `05-backoffice-admin` do ecossistema HSC.

Este documento existe para:

- transformar os contratos reconciliados de `news` em uma feature administrativa implementável
- definir a fronteira funcional das telas administrativas do domínio
- orientar a modelagem frontend da feature de forma previsível e disciplinada
- registrar o estado mínimo da feature para MVP operacional
- reduzir ambiguidade entre contrato HTTP, fluxo de UI, estado local e lifecycle do recurso
- orientar como a feature deve evoluir quando novos contratos do domínio forem adicionados

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Backoffice Admin](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Documentos diretamente relacionados
- [News Admin API Contracts](./news-admin-api-contracts.md)
- [News Admin Integration and Evolution](./news-admin-integration-and-evolution.md)
- [News Functional Smoke Guide](./news-functional-smoke-guide.md)
- [Frontend Structure](./backoffice-admin-frontend-structure.md)
- [Auth, RBAC and Guards](./auth-rbac-and-guards.md)
- [Admin API Contracts](./admin-api-contracts.md)
- [Backoffice Admin Operational Runbooks](./backoffice-admin-operational-runbooks.md)

### Relações com outros contextos
- [Auth API Operations](../04-infra-aws-lightsail/auth-api-operations.md)
- [Portal Estático](../03-portal-estatico/README.md)
- [Documentation System](../00-governance/documentation-system.md)

---

## Escopo

Este documento cobre:

- a feature administrativa de `news` na SPA do Backoffice
- a fronteira funcional entre listagem, criação, edição e mutações de lifecycle
- as rotas administrativas mínimas da feature
- a modelagem recomendada de pastas, serviços, DTOs e estado local
- o comportamento esperado de UX operacional para sucesso, erro e transições de estado
- critérios canônicos de aceitação do MVP de `news`
- o procedimento documental correto para expansão futura da feature

Este documento não cobre em profundidade:

- implementação final de HTML, CSS ou design system
- decisões visuais de layout detalhadas
- SQL, migrations ou operação interna do backend
- implementação concreta de cada componente Angular linha a linha
- o contrato público do Portal além do necessário para validar publish e unpublish

---

## Leitura correta da feature

A feature `news` do Backoffice não é um CRUD isolado sem contexto.

Ela deve ser lida como a materialização administrativa do seguinte fluxo:

1. operador autenticado entra na SPA administrativa
2. a feature consome superfícies `/admin/news*`
3. a Auth API permanece autoridade final de validação, persistência e transição de estado
4. `publish` expõe o item na superfície pública canônica `/content/news*`
5. o Portal público consome o mirror same-origin correspondente

Regra importante:

- a feature do Backoffice não deve falar diretamente com o mirror do Portal
- a feature do Backoffice não deve inventar estados fora dos contratos reconciliados
- `publish` e `unpublish` devem continuar sendo ações explícitas de domínio, não side effect de edição

---

## Estado atual reconciliado que governa a implementação

No checkpoint atual, o runtime real já confirmou:

- `GET /admin/news` protegido por sessão administrativa válida
- `GET /admin/news/:id` protegido por sessão administrativa válida, com detalhe completo e `content`
- `POST /admin/news` com body mínimo `slug`, `title`, `content`
- `PATCH /admin/news/:id`
- `POST /admin/news/:id/publish`
- `POST /admin/news/:id/unpublish`
- `DELETE /admin/news/:id`
- `GET /content/news`
- `GET /content/news/:slug`

Também já está reconciliado que:

- o estado inicial é `draft`
- `publish` preenche `published_at` em UTC
- `unpublish` remove o item da superfície pública
- `delete` remove o item da superfície administrativa

Restrições relevantes para a implementação:

- a Auth API PROD real está publicada na AWS Lightsail e foi atualizada para o commit `728299e`
- a edição do Backoffice deve usar `GET /admin/news/:id` como fonte primária de detalhe
- a edição não deve depender de `editableDrafts`, cache da listagem ou estado roteado como fonte primária
- refresh em `/news/:id/edit` e deep link em nova aba devem funcionar sem estado local prévio

---

## Objetivo funcional do MVP da feature

O MVP de `news` no Backoffice precisa entregar uma feature administrativa pequena, operacional e previsível.

A feature deve permitir:

- visualizar a lista administrativa de notícias
- criar um draft de notícia
- editar um draft ou item já existente
- publicar uma notícia
- despublicar uma notícia
- remover uma notícia
- exibir feedback operacional claro após cada mutação

A feature não precisa, neste estágio, depender de:

- paginação administrativa
- filtros avançados
- busca full text
- autosave
- preview visual sofisticado
- upload integrado de imagem
- editor rich text avançado

Regra importante:

- primeiro entregar previsibilidade operacional do domínio
- depois sofisticar ergonomia

---

## Superfície funcional da feature

### Rotas administrativas mínimas

As rotas mínimas da feature devem ser:

```text
/news
/news/new
/news/:id/edit
```

### Papel de cada rota

#### `/news`

Função:

- listar itens administrativos
- mostrar estado do item
- permitir navegação para criação e edição
- disparar publish, unpublish e delete

#### `/news/new`

Função:

- criar um novo draft de notícia
- validar campos mínimos antes do envio
- persistir via `POST /admin/news`

#### `/news/:id/edit`

Função:

- carregar por `GET /admin/news/:id` e editar um item existente
- permitir update de `title` e `content`
- permitir publish, unpublish e delete no contexto do item, se a UX local fizer sentido

Leitura importante:

- `GET /admin/news/:id` é a fonte primária de hidratação da tela
- a listagem pode acelerar navegação ou atualizar metadados, mas não deve substituir o detalhe completo com `content`
- `missing-draft` não é comportamento esperado para refresh ou deep link

---

## Matriz de telas versus contratos

| Tela / Ação | Contrato principal | Resultado esperado |
|---|---|---|
| `/news` abrir | `GET /admin/news` | lista administrativa carregada |
| `/news/new` salvar | `POST /admin/news` | draft criado |
| `/news/:id/edit` abrir | `GET /admin/news/:id` | detalhe administrativo completo carregado |
| `/news/:id/edit` salvar | `PATCH /admin/news/:id` | item atualizado |
| publicar item | `POST /admin/news/:id/publish` | `status = published` e `published_at` preenchido |
| despublicar item | `POST /admin/news/:id/unpublish` | `status = draft` e `published_at = null` |
| remover item | `DELETE /admin/news/:id` | item removido da lista |
| validar efeito público | `GET /content/news` e `GET /content/news/:slug` | item aparece ou some conforme publish / unpublish |

---

## Modelo canônico de dados para a feature

### DTO de listagem administrativa

```ts
export type AdminNewsListItem = {
  id: number;
  slug: string;
  title: string;
  excerpt: string | null;
  image_url: string | null;
  status: 'draft' | 'published' | string;
  published_at: string | null;
  created_at: string;
  updated_at: string;
};
```

### DTO de detalhe administrativo

```ts
export type AdminNewsDetailItem = AdminNewsListItem & {
  content: string;
};
```

### Envelope do detalhe administrativo

```ts
export type AdminNewsDetailResponse = {
  ok: true;
  item: AdminNewsDetailItem;
};
```

### Envelope da listagem administrativa

```ts
export type AdminNewsListResponse = {
  ok: true;
  count: number;
  items: AdminNewsListItem[];
};
```

### Payload mínimo de criação

```ts
export type CreateNewsPayload = {
  slug: string;
  title: string;
  content: string;
};
```

### Payload seguro já reconciliado para update

```ts
export type UpdateNewsPayload = {
  title?: string;
  content?: string;
};
```

### Resposta de criação mínima observada

```ts
export type CreateNewsResponse = {
  ok: true;
  id: number;
  slug: string;
  status: 'draft' | string;
};
```

### Resposta típica de mutação com item

```ts
export type AdminNewsMutationResponse = {
  ok: true;
  item: AdminNewsListItem;
};
```

Regra importante:

- não promover `excerpt`, `image_url` ou mutação de `slug` como requisito de MVP sem reconciliação adicional do backend

---

## Estrutura recomendada da feature no frontend

A feature `news` deve ser organizada como domínio próprio dentro da SPA.

Estrutura recomendada:

```text
src/app/features/news/
  data-access/
    news-admin-api.service.ts
    news-admin.models.ts
    news-admin.store.ts
  pages/
    news-list-page/
    news-create-page/
    news-edit-page/
  components/
    news-form/
    news-table/
    news-status-badge/
    news-actions/
  utils/
    news-form.mapper.ts
    news-error.mapper.ts
```

### Papel de cada camada

#### `data-access/`

Responsável por:

- contratos HTTP
- DTOs
- estado local da feature
- revalidação após mutações

#### `pages/`

Responsável por:

- composição de tela
- integração entre store e componentes visuais
- leitura de route params e navegação

#### `components/`

Responsável por:

- blocos de UI reutilizáveis da feature
- formulário
- tabela
- ações e badges de estado

#### `utils/`

Responsável por:

- mapeamentos leves
- adaptação de erro do backend para mensagens legíveis na UI

---

## Serviço de domínio recomendado

A feature deve expor um serviço de domínio fino e explícito.

Estrutura recomendada:

```text
features/news/data-access/news-admin-api.service.ts
```

Operações mínimas:

```ts
list(): Observable<AdminNewsListResponse>
detail(id: number): Observable<AdminNewsDetailResponse>
create(payload: CreateNewsPayload): Observable<CreateNewsResponse>
update(id: number, payload: UpdateNewsPayload): Observable<AdminNewsMutationResponse>
publish(id: number): Observable<AdminNewsMutationResponse>
unpublish(id: number): Observable<AdminNewsMutationResponse>
remove(id: number): Observable<{ ok: true; deleted: number }>
```

Regras de implementação:

- usar `withCredentials` na integração HTTP, conforme o modelo session-first do admin
- manter o serviço focado em IO com a Auth API
- centralizar nele apenas o necessário para o domínio
- não espalhar URLs do backend pela UI

---

## Estratégia de estado local da feature

A feature deve separar leitura de coleção e leitura de detalhe.

Estratégia recomendada:

- manter uma store local da feature com a lista administrativa já carregada
- carregar `/news/:id/edit` por `GET /admin/news/:id`
- derivar seletores simples para loading, empty state e item por `id`
- após mutações, revalidar a lista ou sincronizar o item mutado de forma explícita

### Fontes primárias de leitura

Para coleção administrativa:

```http
GET /admin/news
```

Para detalhe administrativo:

```http
GET /admin/news/:id
```

Regra importante:

- a listagem não deve ser a fonte primária de `content` na edição
- `editableDrafts`, cache da listagem e estado roteado podem existir como otimização local, mas não como contrato funcional necessário
- refresh em `/news/:id/edit` e deep link em nova aba devem carregar o detalhe por `id`

---

## Especificação da tela `/news`

## Objetivo funcional

Ser a superfície principal de operação do domínio `news` no admin.

## Conteúdo mínimo da tela

- cabeçalho da feature
- ação primária “nova notícia”
- tabela ou lista administrativa
- colunas mínimas: `title`, `slug`, `status`, `published_at`, `updated_at`
- ações por linha: editar, publicar, despublicar, remover

## Estados mínimos da tela

### Loading

Exibir carregamento enquanto `GET /admin/news` está em andamento.

### Empty

Quando `count = 0`:

- exibir estado vazio claro
- manter CTA de criação visível

### Success

Quando houver itens:

- exibir lista consistente com o contrato administrativo
- mostrar claramente o estado `draft` ou `published`

### Error

Quando a leitura falhar:

- exibir mensagem operacional legível
- permitir retry explícito

---

## Especificação da tela `/news/new`

## Objetivo funcional

Criar um novo draft de notícia com o menor atrito operacional possível.

## Campos mínimos obrigatórios

- `slug`
- `title`
- `content`

## Regras de comportamento

- a UI pode aplicar validação client-side básica de required
- o backend permanece autoridade final de validação
- em sucesso, o frontend deve obter o `id` criado
- após criação, a navegação pode seguir para `/news/:id/edit` ou revalidar a lista, desde que o comportamento seja consistente

## Erros esperados a tratar

- `401 Unauthorized` quando a sessão expirar
- `400 missing_fields` quando a UI enviar body inválido
- conflitos futuros de `slug` quando o backend passar a expor esse erro de forma explícita

---

## Especificação da tela `/news/:id/edit`

## Objetivo funcional

Carregar o detalhe administrativo completo por `id`, editar o conteúdo e permitir operações de lifecycle do recurso.

## Campos mínimos deste checkpoint

- `title`
- `content`

## Campos lidos, mas não obrigatoriamente editáveis neste estágio

- `slug`
- `status`
- `published_at`
- `created_at`
- `updated_at`

## Regras importantes

- a tela deve depender de `GET /admin/news/:id` para carregar `content`
- a tela deve tratar erro real de detalhe indisponível ou item inexistente de modo legível
- a UI não deve mostrar `missing-draft` como estado esperado de refresh ou deep link
- a listagem local não deve substituir o contrato de detalhe

## Ações possíveis na tela

- salvar edição
- publicar
- despublicar
- remover

---

## Especificação do formulário de `news`

O formulário deve ser simples e subordinado ao domínio.

### Campos mínimos do formulário

```text
slug
title
content
```

### Regras de uso por tela

#### Criação

- `slug` editável
- `title` editável
- `content` editável

#### Edição

No checkpoint atual, a decisão mais segura é:

- permitir edição de `title`
- permitir edição de `content`
- tratar `slug` como somente leitura até reconciliação explícita de mutabilidade

Motivo:

- a mutabilidade de `slug` ainda não foi confirmada no runtime reconciliado
- o `slug` participa da identidade pública do conteúdo

---

## Estratégia de UX para mutações

As mutações da feature precisam ser operacionalmente legíveis.

### Após create

- informar sucesso
- persistir o `id` recém-criado no fluxo de navegação
- permitir continuidade natural para edição

### Após update

- refletir imediatamente `updated_at` quando o backend devolver o item completo
- manter a tela consistente com o estado retornado

### Após publish

- atualizar badge de estado para `published`
- refletir `published_at`
- permitir ação de despublicar

### Após unpublish

- atualizar badge de estado para `draft`
- limpar `published_at`
- permitir ação de publicar

### Após delete

- remover o item do estado local apenas após sucesso
- redirecionar para `/news` quando a exclusão ocorrer na tela de edição

---

## Estratégia de tratamento de erro

O tratamento de erro deve ser claro, pequeno e orientado à operação.

### Categorias mínimas

#### Sessão inválida

Sinais típicos:

- `401 Unauthorized`

Comportamento esperado:

- informar que a sessão administrativa expirou ou não é válida
- deixar a camada global de auth tratar redirecionamento quando aplicável

#### Validação

Sinais típicos:

- `400 missing_fields`

Comportamento esperado:

- apontar campos obrigatórios ausentes
- preservar dados já digitados no formulário

#### Falha genérica de backend ou rede

Comportamento esperado:

- mensagem curta e operacional
- ação de retry quando couber

Regra importante:

- a feature não deve inventar semânticas de erro que o backend não devolveu

---

## Guardas e autorização

A feature `news` depende da mesma espinha dorsal de auth do Backoffice.

Leitura correta:

- a feature deve existir apenas sob shell autenticado administrativo
- as rotas `/news*` devem ficar atrás do mesmo modelo de guards já adotado no Backoffice
- a UI não substitui autorização do backend

Consequência:

- mesmo com guarda de rota no frontend, toda mutação continua precisando tratar `401` ou `403`

---

## Critérios canônicos de aceitação do MVP

A feature `news` no Backoffice deve ser considerada pronta neste estágio quando os seguintes critérios forem verdadeiros:

1. `/news` lista corretamente itens de `GET /admin/news`
2. `/news/new` cria item com `slug`, `title` e `content`
3. `/news/:id/edit` carrega detalhe completo por `GET /admin/news/:id`
4. refresh em `/news/:id/edit` funciona
5. deep link em nova aba para `/news/:id/edit` funciona
6. o item pode ser editado via `PATCH /admin/news/:id`
7. `publish` altera o estado administrativo para `published`
8. `unpublish` retorna o item para `draft`
9. `delete` remove o item e a UI reflete isso sem inconsistência operacional
10. a feature trata sessão expirada e erro de validação de forma legível
11. a implementação não depende de cache/listagem/editable drafts como fonte primária de edição

---

## Não objetivos deste checkpoint

Os itens abaixo não fazem parte do objetivo real desta etapa, salvo reconciliação posterior explícita:

- preview público embutido na SPA
- editor rico WYSIWYG
- versionamento de conteúdo
- agendamento de publicação
- paginação server-side
- filtros avançados
- upload de mídia no próprio domínio `news`
- workflow editorial multiestado além de `draft` e `published`

---

## Como adicionar novos contratos ao domínio `news`

A evolução do domínio deve seguir ordem disciplinada.

### Quando um novo contrato surgir

Exemplos:

- mutação explícita de `slug`
- suporte a `excerpt`
- suporte a `image_url`
- paginação, filtro ou busca
- preview administrativo

### Ordem correta de atualização documental

1. atualizar `news-admin-api-contracts.md` com o contrato reconciliado
2. atualizar `news-admin-integration-and-evolution.md` com o novo uso correto do domínio
3. atualizar este documento para refletir o impacto na feature
4. atualizar `README.md` e `backoffice-admin-references-inventory.md` se a nova superfície alterar a navegação ou a fronteira do contexto
5. registrar impl-log apenas quando houver mudança material implementada no runtime ou no código

Regra importante:

- primeiro documentar o contrato reconciliado
- depois expandir a feature sobre base estável

---

## Relação com os demais canônicos do contexto

Este documento complementa:

- `news-admin-api-contracts.md` como fonte de contrato do domínio
- `news-admin-integration-and-evolution.md` como fonte de lifecycle e expansão disciplinada
- `backoffice-admin-frontend-structure.md` como fonte estrutural da SPA
- `auth-rbac-and-guards.md` como fonte de acesso e proteção

Este documento não substitui nenhum deles.

Ele existe para fazer a ponte entre:

- contrato reconciliado
- estrutura frontend
- entrega de feature real

---

## Resumo executivo

A feature `news` do Backoffice Admin já possui base contratual suficiente para operar o MVP administrativo.

A forma correta de implementá-la neste checkpoint é:

- usar `GET /admin/news` como leitura de coleção
- usar `GET /admin/news/:id` como leitura primária da tela de edição
- criar a feature com rotas `/news`, `/news/new` e `/news/:id/edit`
- tratar `create`, `update`, `publish`, `unpublish` e `delete` como operações explícitas de domínio
- não depender de cache/listagem/editable drafts como fonte primária de edição
- manter a implementação pequena, operacional e rigidamente alinhada aos contratos canônicos já confirmados
