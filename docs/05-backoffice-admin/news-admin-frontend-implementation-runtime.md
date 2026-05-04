# News Admin Frontend Implementation Runtime

## Objetivo

Documentar a implementação técnica real do MVP da feature administrativa de `news` no contexto `05-backoffice-admin` do ecossistema HSC.

Este documento existe para:

- registrar a materialização real da feature `news` na SPA administrativa
- substituir ambiguidade entre especificação de implementação e estado efetivamente entregue
- descrever as rotas reais, a topologia de arquivos e a modelagem frontend aplicada no runtime
- registrar a integração HTTP real da feature com a Auth API administrativa
- consolidar a estratégia implementada para estado local, edição e lifecycle do recurso
- registrar a validação funcional já executada em produção
- orientar manutenção futura da feature sem depender de memória informal

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Backoffice Admin](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Documentos diretamente relacionados
- [Backoffice Admin Frontend Structure](./backoffice-admin-frontend-structure.md)
- [Admin API Contracts](./admin-api-contracts.md)
- [News Admin API Contracts](./news-admin-api-contracts.md)
- [News Admin Integration and Evolution](./news-admin-integration-and-evolution.md)
- [News Admin Feature Implementation Spec](./news-admin-feature-implementation-spec.md)
- [News Functional Smoke Guide](./news-functional-smoke-guide.md)
- [Backoffice Admin References Inventory](./backoffice-admin-references-inventory.md)
- [Backoffice Admin Operational Runbooks](./backoffice-admin-operational-runbooks.md)

### Relações com outros contextos
- [Infra AWS Lightsail](../04-infra-aws-lightsail/README.md)
- [Auth API Operations](../04-infra-aws-lightsail/auth-api-operations.md)
- [Portal Estático](../03-portal-estatico/README.md)
- [Documentation System](../00-governance/documentation-system.md)

---

## Escopo

Este documento cobre:

- a implementação real do MVP da feature `news` no frontend administrativo
- as rotas administrativas efetivamente materializadas
- a estrutura de arquivos criada em `src/app/features/news`
- a estratégia implementada de `data-access`, `store`, componentes e páginas
- a integração HTTP real com a Auth API administrativa
- a estratégia efetivamente aplicada para edição usando `GET /admin/news/:id`
- os estados de loading, erro, vazio e mutação já implementados
- a validação funcional em produção concluída para o lifecycle do recurso

Este documento não cobre em profundidade:

- o contrato público completo de `content/news`
- detalhes internos de backend, SQL ou migrations
- design system final, refinamento visual ou acessibilidade aprofundada
- paginação administrativa, filtros avançados, busca textual ou rich text editor
- futuras expansões de `excerpt`, `image_url` ou mutabilidade de `slug`

Esses tópicos vivem em documentos adjacentes ou permanecem fora do escopo do checkpoint atual.

---

## Estado atual

O estado atual reconciliado desta implementação é:

- a feature `news` deixou de ser placeholder em `app.routes.ts`
- a SPA agora expõe rotas administrativas reais para listagem, criação e edição
- a integração com a Auth API usa `withCredentials: true` em todas as mutações e leituras do domínio
- a feature foi organizada sob `src/app/features/news`
- a camada de IO foi separada em models, serviço HTTP e store local
- a edição usa `GET /admin/news/:id` como superfície canônica publicada para carregar detalhe completo com `content`
- refresh em `/news/:id/edit` funciona
- deep link ou nova aba para `/news/:id/edit` funciona
- o ciclo funcional PROD foi validado para carregar edição, editar, publicar, despublicar e remover
- a Auth API PROD real está na AWS Lightsail e foi atualizada para o commit `728299e`

Regra importante:

- este documento registra a implementação real entregue no frontend
- ele não substitui os contratos do domínio nem a especificação de evolução futura

---

## Source of truth

Este documento é canônico para:

- a implementação real do MVP de `news` no frontend do Backoffice
- a estrutura de arquivos materializada na feature
- a modelagem real aplicada no runtime administrativo
- a forma como o frontend usa a leitura por `id` reconciliada neste checkpoint
- a validação funcional executada em PROD sobre a feature

A fonte de verdade deste documento é composta por:

- o código materializado em `src/app/app.routes.ts`
- o código materializado em `src/app/features/news/**`
- os contratos canônicos já documentados em `news-admin-api-contracts.md`
- a especificação canônica de implementação em `news-admin-feature-implementation-spec.md`
- a regra de integração e evolução em `news-admin-integration-and-evolution.md`
- a validação PROD já executada sobre navegação, deep link e lifecycle funcional do domínio

---

## Leitura correta deste documento

Este documento não deve ser lido como spec futura.

Ele deve ser lido como fotografia técnica da implementação real entregue no frontend administrativo para o MVP de `news`.

Leitura correta:

1. os contratos e limites do domínio continuam sendo governados por `news-admin-api-contracts.md`
2. a intenção e o alvo funcional do MVP continuam sendo governados por `news-admin-feature-implementation-spec.md`
3. este documento registra como a feature foi efetivamente materializada na SPA
4. futuras mudanças relevantes devem atualizar este documento quando alterarem topologia, fluxos ou invariantes reais da implementação

---

## Fronteira funcional implementada

O MVP implementado entrega as seguintes superfícies administrativas:

- listagem administrativa em `/news`
- criação de draft em `/news/new`
- edição e lifecycle em `/news/:id/edit`

As seguintes ações já estão materializadas no frontend:

- listar itens administrativos
- criar draft
- editar `title` e `content`
- publicar
- despublicar
- remover
- carregar detalhe administrativo por `GET /admin/news/:id`

A implementação não materializa, neste checkpoint:

- paginação
- filtros
- ordenação explícita configurável
- busca textual server-side
- upload de imagem
- mutação forte de `slug`
- edição de `excerpt`
- edição de `image_url`

---

## Rotas reais implementadas

O placeholder anterior de `news` foi substituído pelas rotas reais abaixo dentro da área protegida do `AdminShellComponent`:

```text
/news
/news/new
/news/:id/edit
```

Títulos configurados:

* `News | HSC Backoffice`
* `Nova News | HSC Backoffice`
* `Editar News | HSC Backoffice`

Leitura importante:

* as rotas permanecem protegidas pelo shell administrativo já governado por `authGuard` e `adminAccessGuard`
* a feature `news` não introduz guard próprio adicional neste checkpoint

---

## Estrutura materializada da feature

A implementação real foi organizada assim:

```text
src/app/features/news/
├── components/
│   ├── news-actions/
│   ├── news-form/
│   ├── news-status-badge/
│   └── news-table/
├── data-access/
│   ├── news-admin-api.service.ts
│   ├── news-admin.models.ts
│   └── news-admin.store.ts
├── pages/
│   ├── news-create-page/
│   ├── news-edit-page/
│   └── news-list-page/
└── utils/
    ├── news-error.mapper.ts
    └── news-form.mapper.ts
```

Leitura correta desta topologia:

* `components/` concentra componentes reutilizáveis da feature
* `data-access/` concentra IO, tipagem administrativa e estado local da feature
* `pages/` concentra as três superfícies reais do MVP
* `utils/` concentra mapeamentos locais pequenos e específicos do domínio

---

## Camada HTTP implementada

A integração real com a Auth API foi materializada em:

```text
src/app/features/news/data-access/news-admin-api.service.ts
```

Operações implementadas no serviço:

* `list()`
* `detail(id)`
* `create(payload)`
* `update(id, payload)`
* `publish(id)`
* `unpublish(id)`
* `remove(id)`

Endpoints consumidos:

```http
GET    /admin/news
GET    /admin/news/:id
POST   /admin/news
PATCH  /admin/news/:id
POST   /admin/news/:id/publish
POST   /admin/news/:id/unpublish
DELETE /admin/news/:id
```

Invariantes adotados na implementação:

* todas as chamadas usam `withCredentials: true`
* a URL base do backend permanece centralizada em `API_BASE_URL`
* a UI não espalha endpoints diretamente em componentes de página
* publish e unpublish permanecem mutações explícitas, não patch de `status`
* a edição carrega `content` via detalhe administrativo por `id`

---

## Modelagem administrativa efetivamente aplicada

Os principais tipos administrativos materializados no frontend foram:

```text
AdminNewsListItem
AdminNewsListResponse
CreateNewsPayload
UpdateNewsPayload
CreateNewsResponse
AdminNewsMutationResponse
AdminNewsDeleteResponse
NewsFormValue
AdminNewsDetailItem
AdminNewsDetailResponse
```

Leitura importante:

* `AdminNewsListItem` representa a leitura administrativa de lista
* `AdminNewsDetailItem` representa a leitura administrativa completa com `content`
* `NewsFormValue` representa o formulário mínimo do domínio no frontend
* `UpdateNewsPayload` permanece limitado a `title` e `content`, em linha com o checkpoint canônico atual

Regra importante:

* a implementação não promove `slug`, `excerpt` ou `image_url` como payload mutável forte do MVP

---

## Store local implementada

A feature materializou estado local em:

```text
src/app/features/news/data-access/news-admin.store.ts
```

Estratégia aplicada:

* Signals como mecanismo de estado local da feature
* lista administrativa como leitura de coleção do domínio
* detalhe administrativo por `id` como leitura primária da edição
* sincronização imediata do item mutado quando a resposta traz `item`
* revalidação da lista após criação

Sinais principais materializados:

* `items`
* detalhe ativo ou estado equivalente de detalhe por `id`
* `loading`
* `loaded`
* `error`
* `activeMutation`

Derivações principais materializadas:

* `count`
* `isEmpty`
* resolução de item por `id`
* resolução de detalhe por `id`

Operações principais da store:

* `load()`
* `refresh()`
* `ensureLoaded()`
* `loadDetail(id)` ou operação equivalente de detalhe
* `ensureItem(id)`
* `create(value)`
* `update(id, payload, formValue?)`
* `publish(id)`
* `unpublish(id)`
* `remove(id)`

---

## Estratégia real para edição com `GET /admin/news/:id`

A implementação foi reconciliada com o contrato PROD atual:

* a leitura dedicada `GET /admin/news/:id` está publicada e reconciliada
* a tela `/news/:id/edit` usa essa superfície para carregar `content`
* refresh direto na rota funciona
* deep link ou abertura em nova aba funciona

A solução materializada no frontend é:

1. tratar `GET /admin/news` como leitura de coleção
2. tratar `GET /admin/news/:id` como fonte primária de detalhe
3. renderizar o formulário de edição apenas após carregar o detalhe completo
4. usar `PATCH /admin/news/:id` para persistir `title` e `content`
5. reconciliar metadados retornados por `update`, `publish` e `unpublish`

Consequência prática:

* a edição completa não depende mais de estado local previamente semeado
* refresh e deep link preservam a experiência funcional
* o estado antigo de `missing-draft` não é comportamento esperado no runtime atual

Regra importante:

* `editableDrafts`, cache da listagem ou estado roteado não devem voltar a ser fonte primária de edição
* a feature opera sobre o contrato real publicado em PROD

---

## Componentes reutilizáveis materializados

A implementação criou os seguintes componentes específicos do domínio:

### `news-status-badge`

Função:

* exibir o estado administrativo do item
* materializar visualmente a distinção entre `draft` e `published`

### `news-actions`

Função:

* concentrar ações por linha de item administrativo
* emitir eventos de `edit`, `publish`, `unpublish` e `remove`

### `news-table`

Função:

* renderizar a tabela administrativa de `/news`
* exibir colunas mínimas do contrato administrativo
* integrar ações por item

### `news-form`

Função:

* centralizar o formulário do domínio para criação e edição
* operar em dois modos: `create` e `edit`
* manter `slug` editável na criação e somente leitura na edição
* exibir metadados quando a tela de edição tiver contexto suficiente

---

## Página `/news` implementada

A página de listagem foi materializada em:

```text
src/app/features/news/pages/news-list-page/
```

Responsabilidades implementadas:

* cabeçalho da feature
* CTA primária “Nova notícia”
* leitura inicial do domínio via store
* retry explícito em caso de erro
* estado vazio com CTA de criação
* renderização de tabela administrativa quando houver itens
* disparo de publish, unpublish e remove por linha
* navegação para criação e edição

Estados implementados:

### Loading

* exibido quando a primeira leitura do domínio está em andamento

### Error

* mensagem operacional legível
* ação explícita de retry

### Empty

* mensagem clara de ausência de itens
* CTA de criação permanece visível

### Success

* renderização da tabela com colunas mínimas e ações por item

---

## Página `/news/new` implementada

A página de criação foi materializada em:

```text
src/app/features/news/pages/news-create-page/
```

Comportamento implementado:

* renderiza `news-form` em modo `create`
* aplica validação client-side básica de required
* usa `store.create(value)` como ação principal
* em sucesso, navega para `/news/:id/edit`
* em erro, exibe mensagem operacional oriunda da store
* ação de cancelamento retorna para `/news`

Leitura importante:

* o backend continua autoridade final de validação
* a continuidade para edição depende do `id` retornado e da leitura de detalhe por `GET /admin/news/:id`

---

## Página `/news/:id/edit` implementada

A página de edição foi materializada em:

```text
src/app/features/news/pages/news-edit-page/
```

Responsabilidades implementadas:

* resolver `id` da rota
* validar `id` inválido
* carregar detalhe por `GET /admin/news/:id`
* renderizar `news-form` em modo `edit` quando houver detalhe administrativo com `content`
* manter ações de lifecycle disponíveis com base no item administrativo carregado
* tratar erro de detalhe indisponível ou item inexistente de forma legível

Estados de resolução implementados:

* `loading`
* `ready`
* `invalid-id`
* `not-found`

Ações implementadas na página:

* salvar alterações
* publicar
* despublicar
* remover
* voltar para listagem

Leitura importante:

* `slug` permanece somente leitura neste checkpoint
* `title` e `content` são os campos mutáveis efetivamente suportados na tela de edição

---

## Tratamento de erro implementado

A feature materializou um mapper local de erro em:

```text
src/app/features/news/utils/news-error.mapper.ts
```

Tratamentos explícitos implementados:

* `401` → sessão administrativa inválida ou expirada
* `403` → falta de permissão
* `400` com `missing_fields` → mensagem operacional com campos obrigatórios
* fallback operacional genérico para demais falhas

Leitura correta:

* a feature não tenta catalogar erros que ainda não foram promovidos como contrato forte do domínio
* o mapper permanece pequeno e operacional

---

## Tratamento de loading, empty e mutação

A implementação já materializa tratamento operacional dos seguintes estados:

### Leitura inicial

* loading explícito
* empty explícito
* erro explícito com retry

### Mutação em andamento

* `activeMutation` como referência de ação corrente
* bloqueio de ações concorrentes relevantes na UI
* feedback textual simples durante processamento administrativo

### Estados da edição

* carregamento de contexto
* detalhe carregado
* item inexistente ou detalhe indisponível

---

## Fluxo funcional efetivamente validado

A implementação foi validada no Backoffice Admin PROD com a Auth API PROD publicada na AWS Lightsail.

Fluxo validado:

1. edição carrega conteúdo por `GET /admin/news/:id`
2. refresh em `/news/:id/edit` funciona
3. deep link ou nova aba para `/news/:id/edit` funciona
4. edição persiste via `PATCH /admin/news/:id`
5. publicação funciona via `POST /admin/news/:id/publish`
6. despublicação funciona via `POST /admin/news/:id/unpublish`
7. remoção funciona via `DELETE /admin/news/:id`
8. CORS administrativo permite `DELETE`

Também foi validado:

* `Access-Control-Allow-Methods: GET,POST,PATCH,DELETE,OPTIONS`
* a superfície pública `/content/news` refletiu a publicação na Auth API
* o fluxo público via ETL Hostinger e Portal Estático exibiu a notícia
* console do Portal Estático sem erro no smoke validado

---

## O que está confirmado

Neste checkpoint, está confirmado que:

* a feature `news` já existe como superfície funcional real no Backoffice
* o placeholder anterior foi removido do roteamento para `news`
* a integração administrativa principal do domínio está operando no frontend
* o MVP PROD já é operacional para o lifecycle completo do recurso
* a solução implementada para edição usa a leitura dedicada por `id` reconciliada

---

## O que este documento deliberadamente não promove

Este documento deliberadamente não promove como verdade canônica consolidada:

* mutabilidade de `slug`
* mutabilidade de `excerpt`
* mutabilidade de `image_url`
* paginação administrativa
* filtros administrativos
* busca textual administrativa
* catálogo completo de erros de domínio

Leitura canônica atual:

* esses pontos continuam fora do escopo forte da implementação entregue
* devem ser documentados apenas quando o contrato e o runtime correspondente forem explicitamente validados

---

## Relação entre este documento e os demais canônicos do domínio

A relação correta entre os documentos de `news` no contexto `05-backoffice-admin` passa a ser:

* `news-admin-api-contracts.md` → contratos reconciliados do domínio
* `news-admin-integration-and-evolution.md` → uso correto e evolução disciplinada do domínio
* `news-admin-feature-implementation-spec.md` → alvo funcional e estrutural do MVP
* `news-admin-frontend-implementation-runtime.md` → implementação real materializada no frontend

Regra importante:

* este documento não substitui os demais
* ele fecha a camada de implementação real dentro da trilha documental do domínio

---

## Limites do documento

Este documento deve ser revisado quando houver mudança relevante em pelo menos um dos pontos abaixo:

* rotas da feature
* topologia de arquivos de `news`
* payloads efetivamente usados pela UI
* estratégia de store local ou edição
* mudança relevante na leitura dedicada por `id`
* expansão forte para novos campos administrativos
* mudança relevante no ciclo de validação funcional do MVP

Este documento não precisa ser reescrito por mudanças pequenas puramente visuais.

---

## Critério de pronto

Este documento pode ser considerado saudável quando:

* registra a implementação real da feature sem depender de memória informal
* explica claramente como a feature foi materializada no frontend
* distingue corretamente contrato, spec e runtime implementado
* deixa explícita a estratégia usada com `GET /admin/news/:id`
* registra o ciclo funcional mínimo já validado no ambiente PROD
* reduz ambiguidade operacional para manutenção futura da feature

---

## Última revisão

* Status: ativo
* Classificação: canônico
* Contexto: `05-backoffice-admin` / implementação real do MVP de `news`
* Última revisão: 2026-05-04
