# References Inventory

## Navegação rápida

- [Home da documentação](../README.md)
- [Backoffice Admin](./README.md)
- [Master Index](../00-governance/99-master-index.md)

---
## Objetivo

Documentar o inventário de referências, dependências, artefatos, integrações e gaps do contexto `05-backoffice-admin` do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- as referências canônicas que governam o Backoffice Admin
- os contextos e documentos dos quais este contexto depende
- os artefatos técnicos previstos e já materializados para a SPA administrativa
- as integrações centrais do Backoffice com o restante do ecossistema
- os recursos e superfícies que já foram reconciliados com o runtime real
- os gaps conhecidos, pendências e zonas de atenção do contexto

---

## Escopo

Este documento cobre:

- referências documentais do Backoffice Admin
- dependências arquiteturais do contexto
- inventário lógico de artefatos da SPA administrativa
- integrações com Auth API e contextos adjacentes
- inventário inicial de domínios administrativos
- pontos de reconciliação com runtime real
- gaps conhecidos de contrato, runtime e operação

Este documento não cobre em profundidade:

- conteúdo completo dos demais documentos
- contratos detalhados endpoint a endpoint
- runbooks operacionais completos
- configuração detalhada de deploy
- detalhes de infraestrutura de host
- implementação exata de banco e backend

Esses tópicos vivem em documentos próprios.

---

## Papel deste documento no contexto

Este arquivo deve funcionar como inventário rápido do contexto `05-backoffice-admin`.

Ele existe para responder perguntas como:

- quais documentos governam este contexto?
- de quais outros contextos o Backoffice depende?
- quais artefatos técnicos a SPA administrativa deve ter?
- quais integrações o produto administrativo consome?
- o que já está reconciliado e o que ainda está em aberto?
- quais domínios já entram no MVP?
- quais pontos ainda exigem confirmação no runtime real?

Regra importante:

- este documento não substitui o `README.md`
- ele complementa o contexto com visão de inventário e reconciliação

---

## Estado atual do contexto

O estado atual conhecido do contexto é:

- o contexto `05-backoffice-admin` já não está apenas em formalização inicial
- o Backoffice Admin é uma SPA administrativa publicada separada do Portal público
- a Auth API é a dependência dinâmica central do contexto
- o desenho reconciliado do frontend assume Angular 21 + TypeScript + Signals
- o modelo de auth do admin é session-first
- a jornada publicada já inclui:
  - `/login`
  - `/auth/callback`
  - `/dashboard`
- o backend permanece autoridade final para autenticação, autorização e invariantes
- `seasons`, `news` e `events` permanecem como domínios iniciais
- `seasons` já possui listagem administrativa inicial implementada no Backoffice
- `seasons` já possui criação administrativa em `draft` implementada no Backoffice
- `seasons` já possui edição administrativa de metadados implementada no Backoffice
- `seasons` já possui ações administrativas de lifecycle implementadas na listagem do Backoffice
- a leitura admin canônica de Seasons já está disponível no backend
- a criação admin canônica de Seasons já está disponível no backend por `POST /admin/seasons`
- a edição admin canônica de Seasons já está disponível no backend por `PATCH /admin/seasons/:slug`
- activate e close de Seasons já estão disponíveis na UI por `POST /admin/seasons/:slug/activate` e `POST /admin/seasons/:slug/close`
- `POST /admin/uploads` já existe como contrato protegido da Auth API para upload de imagens administrativas
- o formulário administrativo de News já integra upload visual de imagem, URL, preview e limpeza de `image_url`, com smoke manual local validado
- o contrato de `cover_image_url` para Seasons já existe na Auth API
- o formulário administrativo de Seasons já integra upload visual de capa, URL, preview e limpeza de `cover_image_url`, com smoke manual local validado
- ETL/Static API v2 e Portal ainda estão pendentes para publicar ou consumir `cover_image_url` nos JSONs estáticos quando aplicável
- a fundação UI Material compartilhada já existe em `src/app/shared/ui` para feedback, confirmação e input simples em fluxos principais de `news`, `seasons` e `users`
- dark mode, toggle de tema, design system completo e padronização visual total ainda permanecem como lacunas futuras
- parte importante da auth base já está reconciliada com runtime real
- parte da expansão de domínio ainda depende de evolução incremental de contrato e implementação

Regra importante:

- este documento deve distinguir claramente:
  - o que já é referência canônica consolidada
  - o que é artefato efetivamente materializado
  - o que ainda é gap ou pendência

---

## Referências canônicas diretas deste contexto

Os documentos canônicos diretos de `05-backoffice-admin` são:

- `docs/05-backoffice-admin/README.md`
- `docs/05-backoffice-admin/backoffice-admin-architecture-runtime.md`
- `docs/05-backoffice-admin/backoffice-admin-frontend-structure.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/news-admin-api-contracts.md`
- `docs/05-backoffice-admin/news-admin-integration-and-evolution.md`
- `docs/05-backoffice-admin/news-admin-feature-implementation-spec.md`
- `docs/05-backoffice-admin/news-admin-frontend-implementation-runtime.md`
- `docs/05-backoffice-admin/seasons-admin-list-functional-smoke-guide.md`
- `docs/05-backoffice-admin/backoffice-ui-material-foundation.md`
- `docs/05-backoffice-admin/backoffice-admin-operational-runbooks.md`
- `docs/05-backoffice-admin/backoffice-admin-references-inventory.md`

Leitura recomendada:

- `README.md` define a fronteira do contexto
- `architecture-runtime.md` define a topologia funcional do produto administrativo
- `frontend-structure.md` define a estrutura da SPA
- `auth-rbac-and-guards.md` define acesso e proteção
- `admin-api-contracts.md` define contratos transversais com a Auth API
- `news-admin-api-contracts.md` consolida o contrato canônico do domínio `news`
- `news-admin-integration-and-evolution.md` define uso e expansão disciplinada do domínio `news`
- `news-admin-feature-implementation-spec.md` traduz o domínio reconciliado em feature implementável no frontend
- `news-admin-frontend-implementation-runtime.md` registra a implementação real materializada no frontend
- `seasons-admin-list-functional-smoke-guide.md` registra a administração funcional de `seasons`, seus contratos de leitura/criação/edição/lifecycle e o smoke funcional
- `backoffice-ui-material-foundation.md` registra a fundação transversal de UI Material para feedback, confirmação e input simples, incluindo lacunas explícitas de tema e padronização visual
- `operational-runbooks.md` define validação e troubleshooting
- este arquivo consolida o inventário de referência e dependência

---

## Referências canônicas adjacentes obrigatórias

Este contexto depende diretamente dos seguintes documentos e contextos adjacentes.

### Governança documental

- `docs/00-governance/README.md`
- `docs/00-governance/documentation-system.md`
- `docs/00-governance/99-master-index.md`

Papel dessas referências:

- governar a existência do contexto
- definir padrão documental
- sustentar reconciliação com runtime real
- garantir manutenção do contexto vivo

---

### Infraestrutura dinâmica / Auth API

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-references-inventory.md`

Papel dessas referências:

- registrar o runtime dinâmico real consumido pelo Backoffice
- sustentar o modelo session-first
- registrar o fluxo real de magic link admin
- sustentar callback, cookies e sessão real
- registrar postura fail-closed em mutações sensíveis

---

### Portal público

- `docs/03-portal-estatico/README.md`
- demais documentos do contexto `03-portal-estatico`, quando relevantes

Papel dessas referências:

- preservar a separação entre público e administrativo
- evitar confusão entre consumo público e gestão administrativa
- manter clareza de fronteira entre Portal e Backoffice

---

### Legado reconciliado

- `docs/98-legacy/HSC_MASTER_DOCUMENTATION.md`
- `docs/98-legacy/HSC_Master_Blueprint.md`

Papel dessas referências:

- sustentar reconciliação histórica
- registrar roadmap e visão anterior de camadas
- reaproveitar material de `news`, `seasons`, `events` e RBAC
- servir como base de comparação, não como autoridade superior ao canônico atual

---

## Referências canônicas novas do domínio `news`

O domínio `news` passa a contar com quatro referências canônicas específicas dentro do contexto `05-backoffice-admin`:

- `docs/05-backoffice-admin/news-admin-api-contracts.md`
- `docs/05-backoffice-admin/news-admin-integration-and-evolution.md`
- `docs/05-backoffice-admin/news-admin-feature-implementation-spec.md`

Papel dessas referências:

- separar contrato transversal de auth dos contratos específicos de domínio
- consolidar payloads, responses e lifecycle já reconciliados de `news`
- orientar a implementação das telas `/news`, `/news/new` e `/news/:id/edit`
- estabilizar a passagem entre contrato reconciliado, spec e implementação real materializada
- registrar a implementação real do MVP no frontend administrativo
- registrar a fronteira entre Backoffice, Auth API e mirror público do Portal
- orientar expansão futura do domínio sem transformar `admin-api-contracts.md` em documento monolítico

---

## Referência canônica inicial do domínio `seasons`

O domínio `seasons` passa a contar com uma referência funcional específica para a administração inicial:

- `docs/05-backoffice-admin/seasons-admin-list-functional-smoke-guide.md`

Papel desta referência:

- registrar Season como ciclo competitivo oficial do servidor HSC
- documentar os contratos admin de leitura e criação de Seasons
- registrar que `/seasons` já é página real do Backoffice
- registrar que `/seasons/new` já cria Seasons em `draft`
- preservar o smoke funcional local
- distinguir a listagem implementada das lacunas de lifecycle, ranking, partidas, Portal e ETL

---

## Referências históricas e de apoio

Além das referências canônicas obrigatórias, este contexto pode depender de materiais históricos ou de implantação que ajudem na reconciliação de runtime.

Exemplos típicos:

- docs de PR ou HSC_IMPL relevantes
- notas de implantação que afetem contratos admin
- documentos de MVP de domínio
- backlog de features administrativas
- registros de gaps encontrados durante implementação
- impl-logs ligados ao checkpoint de auth admin magic link e cutover de migrations

Regra importante:

- documentos de apoio não substituem o contexto canônico
- quando um documento de apoio alterar entendimento estrutural, o contexto 05 deve ser atualizado

---

## Inventário lógico da aplicação Backoffice

A SPA administrativa prevista por este contexto deve conter, no mínimo, os seguintes artefatos lógicos.

### Entrada e configuração

- `src/main.ts`
- `src/app/app.component.ts`
- `src/app/app.config.ts`
- `src/app/app.routes.ts`

Papel:

- bootstrap Angular 21 standalone
- providers globais
- roteamento principal
- root component da aplicação

---

### Core

Estruturas previstas:

- `src/app/core/auth/`
- `src/app/core/guards/`
- `src/app/core/interceptors/`
- `src/app/core/layout/`
- `src/app/core/config/`
- `src/app/core/services/`
- `src/app/core/models/`

Papel:

- infraestrutura transversal
- auth administrativa
- estado de sessão
- guards
- interceptors
- layout principal do shell
- configuração global da aplicação

---

### Shared

Estruturas previstas:

- `src/app/shared/ui/`
- `src/app/shared/table/`
- `src/app/shared/forms/`
- `src/app/shared/feedback/`
- `src/app/shared/utils/`
- `src/app/shared/types/`

Papel:

- reaproveitamento controlado
- componentes transversais
- tabela e formulário base
- feedback visual
- feedback transitório e persistente com base Material
- confirmação e input simples padronizados
- helpers utilitários
- tipos neutros

---

### Features

Slices iniciais previstos:

- `src/app/features/auth/`
- `src/app/features/dashboard/`
- `src/app/features/seasons/`
- `src/app/features/news/`
- `src/app/features/events/`

Estrutura mínima por feature:

- `pages/`
- `components/`
- `services/`
- `store/`
- `models/`

Papel:

- organizar o admin por domínio
- manter baixo acoplamento
- preservar fronteiras claras entre features

---

## Inventário funcional de domínios administrativos

## Auth

Papel esperado:

- sustentar a jornada real de login do Backoffice
- materializar `/login`
- materializar `/auth/callback`
- revalidar sessão e entregar `/dashboard`

Superfícies administrativas reais ou esperadas:

- login por magic link
- callback com revalidação de sessão
- transição para dashboard autenticado

Status do domínio no contexto:

- auth base já reconciliada com runtime real
- ainda pode evoluir em ergonomia e refinamento

---

## Dashboard

Papel esperado:

- entrada autenticada do Backoffice
- validação de shell, sessão e navegação
- superfície técnica inicial
- não precisa nascer como dashboard analítico sofisticado

Status do domínio no contexto:

- domínio estrutural do produto
- prioridade alta para o shell inicial
- superfície já real no runtime publicado

---

## Seasons

Papel esperado:

- domínio administrativo prioritário do MVP
- representar ciclos competitivos oficiais do servidor HSC
- validar listagem administrativa inicial
- validar criação administrativa inicial em `draft`
- validar edição administrativa inicial de metadados
- operar lifecycle administrativo básico de ativação e fechamento
- manter invariantes de domínio sob autoridade do backend

Superfícies administrativas disponíveis na UI atual:

- listagem admin
- criação admin em `draft`
- edição admin de metadados
- activate
- close

Status do domínio no contexto:

- listagem administrativa inicial implementada
- criação administrativa em `/seasons/new` implementada
- edição administrativa em `/seasons/:slug/edit` implementada
- leitura admin canônica disponível no backend por `GET /admin/seasons` e `GET /admin/seasons/:slug`
- criação admin canônica disponível no backend por `POST /admin/seasons`
- edição admin canônica disponível no backend por `PATCH /admin/seasons/:slug`
- lifecycle admin canônico disponível no backend por `POST /admin/seasons/:slug/activate` e `POST /admin/seasons/:slug/close`
- `cover_image_url` disponível no backend e no formulário administrativo de Seasons
- rota `/seasons` materializada como página funcional do Backoffice
- rota `/seasons/new` materializada como página funcional do Backoffice
- rota `/seasons/:slug/edit` materializada como página funcional do Backoffice
- ações `Ativar` e `Fechar` materializadas na listagem `/seasons`
- ranking por season, partidas associadas, snapshot histórico, Portal, ETL e propagação de `cover_image_url` na Static API v2 seguem como lacunas futuras

---

## News

Papel esperado:

- domínio editorial administrativo
- validar listagem, draft/edit e ações sensíveis
- sustentar publish/unpublish/delete

Superfícies administrativas esperadas:

- listagem admin
- create/edit
- publish
- unpublish
- delete

Status do domínio no contexto:

- maduro
- forte candidato para segunda feature principal após `seasons`

---

## Events

Papel esperado:

- domínio de agenda administrativa
- criar, editar e remover/cancelar eventos
- separar gestão admin de interação pública de presença

Superfícies administrativas esperadas:

- listagem admin
- detail/edit admin
- create
- delete ou cancelamento

Status do domínio no contexto:

- previsto
- importante para MVP ampliado
- ainda pede reconciliação maior de contrato e fronteira com fluxos públicos

---

## Inventário de auth e acesso

A camada de acesso prevista para o Backoffice deve conter, no mínimo:

### Auth store transversal

Artefato esperado:

- `src/app/core/auth/auth-session.store.ts`

Papel:

- resolver sessão
- manter identidade atual
- expor papel atual
- alimentar guards e UI derivada
- oferecer `reloadSession()` para bootstrap, callback e rechecagem explícita

---

### Serviço de integração auth

Artefato esperado:

- `src/app/core/auth/auth-api.service.ts`

Papel:

- chamadas de sessão
- request de magic link
- apoio a callback e revalidação
- ponte entre frontend e contrato de auth

---

### Guards

Artefatos esperados:

- `src/app/core/guards/auth.guard.ts`
- `src/app/core/guards/admin-access.guard.ts`
- `src/app/core/guards/role.guard.ts`

Papel:

- proteger rotas
- separar autenticação de autorização mínima
- suportar rotas sensíveis por papel

---

### Interceptors

Artefatos esperados:

- `src/app/core/interceptors/auth.interceptor.ts`
- `src/app/core/interceptors/auth-error.interceptor.ts`

Papel:

- comportamento transversal de requests
- tratamento consistente de 401, quando apropriado
- alinhamento entre resposta HTTP e estado da sessão

---

## Inventário de integração com a Auth API

A integração principal do Backoffice é com a Auth API.

Classes de integração previstas:

### Sessão e identidade

Objetivo:

- descobrir se há sessão
- resolver operador atual
- obter papel efetivo

Status:

- já reconciliado em runtime real
- `GET /auth/session` já é contrato real do produto administrativo

---

### Login admin por magic link

Objetivo:

- iniciar acesso administrativo real
- solicitar link mágico
- consumir callback publicado
- validar sessão após o callback

Status:

- já reconciliado em runtime real
- depende de email real e callback publicado

---

### Leitura administrativa

Objetivo:

- popular listagens
- carregar formulários
- obter estados de domínio

Status:

- `news` e parte de `seasons` têm base mais madura
- leituras admin completas ainda podem exigir confirmação por evidência funcional

---

### Mutação administrativa

Objetivo:

- criar
- editar
- publicar
- ativar
- fechar
- excluir
- cancelar

Status:

- núcleo do valor do Backoffice
- writes sensíveis devem respeitar fail-closed e auditoria

---

## Inventário de respostas e erros relevantes

O Backoffice deve estar preparado para lidar com estas classes de resposta:

- `200` / `201` / sucesso equivalente
- `401` não autenticado
- `403` sem permissão
- `404` recurso ausente
- `409` conflito de domínio/invariante
- `422` ou equivalente de validação
- `5xx` falha sistêmica/backend

Além disso, no fluxo de callback de auth, o frontend deve reconhecer explicitamente:

- `status=ok`
- `invalid_or_expired_link`
- `consume_failed`
- `missing_token`
- `forbidden`

Papel deste inventário:

- orientar stores, guards e UI
- garantir que o produto trate estados reais
- evitar que erro de domínio seja tratado como erro genérico

---

## Inventário de artefatos de UI esperados

Componentes e estruturas transversais que o Backoffice deve prever:

### Shell

- `admin-shell`
- `sidebar`
- `header`
- `page-container`

### Feedback

- loader de página
- bloco de erro
- empty state
- confirmação de ação destrutiva
- feedback simples de sucesso

### Tabelas

- tabela administrativa base
- ações por linha
- suporte a loading, vazio e erro

### Formulários

- formulários por domínio
- validação básica
- submit controlado
- hidratação de edição

Regra importante:

- esses artefatos devem nascer para suportar domínios reais
- não como abstrações genéricas prematuras

---

## Inventário de estados críticos do frontend

O Backoffice deve tratar, no mínimo, estes estados críticos.

### Estado de aplicação

- boot da aplicação
- shell carregado
- erro fatal de bootstrap

### Estado de sessão

- unknown
- authenticated
- unauthenticated
- error

### Estado de callback

- callback carregando
- callback validado
- callback inválido ou expirado
- callback com falha técnica

### Estado de listagem

- loading
- success
- empty
- error

### Estado de formulário

- idle
- loading initial data
- dirty
- submitting
- success
- error

### Estado de ação sensível

- disponível
- indisponível por papel
- em confirmação
- executando
- sucesso
- rejeitada por permissão
- rejeitada por invariante
- falha sistêmica

---

## Inventário de dependências técnicas da stack

A stack técnica alvo do Backoffice, neste contexto, é:

- Angular 21
- TypeScript
- standalone components
- Angular Router
- HttpClient
- Signals
- RxJS

Leitura dessa stack:

- Angular 21 é a base da SPA
- TypeScript é a linguagem de implementação
- standalone é a estratégia estrutural
- Signals é a estratégia primária de estado local e derivado
- RxJS permanece complementar para HTTP e fluxos assíncronos

---

## Inventário de princípios técnicos a preservar

Este contexto depende da preservação dos seguintes princípios:

- separação entre Backoffice e Portal
- backend como autoridade final
- session-first
- break-glass só como contingência
- fail-closed para writes sensíveis
- UI organizada por domínio
- stores locais por feature
- abstração moderada no MVP
- documentação viva e reconciliada com runtime

---

## Inventário de superfícies conhecidas e superfícies-alvo

Este contexto deve separar explicitamente:

### Superfícies conhecidas

Superfícies já reconciliadas com maior clareza no runtime atual.

Exemplos:

- `/login`
- `/auth/callback`
- `/dashboard`
- `GET /auth/session`
- `POST /auth/magic-link/request`
- trilha administrativa de `news`
- listagem, criação, edição e lifecycle básico administrativo de `seasons`

### Superfícies-alvo

Superfícies necessárias para o Backoffice operar bem, mas ainda sujeitas a reconciliação fina por evidência funcional.

Exemplos:

- ranking, partidas e integrações futuras de `seasons`
- leitura admin canônica de `events`
- lifecycle final de `events` no admin
- refinamento final da fronteira entre staff flow e user flow em `events`

Regra importante:

- o documento deve evitar tratar superfície-alvo como superfície já materializada

---

## Inventário de gaps conhecidos

Os principais gaps atuais do contexto são:

- ranking, partidas e integrações futuras de `seasons` ainda pedem reconciliação própria
- leituras admin completas de `events` ainda pedem consolidação contratual fina
- `events` ainda exige reconciliação melhor entre fluxo administrativo e fluxo público
- troubleshooting de frontend ainda depende fortemente de console/network + logs da Auth API
- o contexto ainda pode amadurecer em observabilidade funcional e consistência de domínio

---

## Inventário de riscos de erosão arquitetural

Os principais riscos de erosão do contexto são:

- Backoffice começar a depender do Portal para mutações
- `shared/` virar repositório genérico sem fronteira
- `core/` absorver lógica de domínio
- `events` nascer misturando staff flow e user flow
- a UI assumir permissão acima do backend
- ações sensíveis usarem optimistic update ingênuo
- docs de contexto ficarem atrás do runtime real

---

## Artefatos que devem nascer cedo

Os artefatos que mais cedo devem existir na implementação são:

### Estruturais

- `main.ts`
- `app.config.ts`
- `app.routes.ts`
- shell administrativo

### Acesso

- `auth-session.store.ts`
- `auth-api.service.ts`
- guards
- interceptors básicos
- `login-page`
- `auth-callback-page`

### Primeiro domínio

- `features/seasons/`
- listagem
- formulário
- ações de lifecycle

Motivo:

- esses artefatos validam a espinha dorsal do Backoffice

---

## Artefatos que podem esperar um pouco mais

Os artefatos que podem nascer em fase seguinte são:

- componentes shared mais sofisticados
- dashboard mais rico
- refinamentos avançados de tabela
- filtros e paginação avançados
- telemetria frontend mais elaborada
- evolução de `events` além do CRUD básico
- abstrações adicionais de store ou UI

Motivo:

- não são necessários para validar o núcleo do MVP

---

## Inventário de critérios de prontidão do contexto

O contexto `05-backoffice-admin` pode ser considerado minimamente pronto quando:

- sua fronteira documental estiver estável
- os contratos principais estiverem registrados
- a estrutura Angular 21 estiver fixada
- a estratégia de auth/RBAC estiver explícita
- os runbooks mínimos estiverem definidos
- os gaps estiverem visíveis e não escondidos
- a auth base publicada já estiver operacional

A implementação do produto pode continuar evoluindo com gaps abertos, desde que:

- os gaps estejam nomeados
- seu impacto esteja entendido
- a ordem de implementação respeite o grau de maturidade de cada domínio

---

## Ordem recomendada de consulta deste inventário

Este documento deve ser consultado:

1. ao abrir o contexto pela primeira vez
2. antes de começar implementação de uma feature admin
3. ao reconciliar contrato com a Auth API
4. ao revisar dependências entre contextos
5. ao identificar gaps de runtime ou documentação
6. ao preparar backlog ou PR incremental

---

## Resumo executivo

O `references-inventory.md` do contexto `05-backoffice-admin` existe para manter uma visão consolidada de:

- referências canônicas
- dependências arquiteturais
- artefatos previstos e materializados da SPA
- integrações com a Auth API
- domínios administrativos do MVP
- gaps e riscos ainda abertos

Ele deve ser lido como o mapa de inventário do Backoffice Admin do HSC:

- amplo o suficiente para orientar o contexto
- estável o suficiente para não virar impl-log disfarçado
- específico o suficiente para reduzir drift documental
