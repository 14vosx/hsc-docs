# Frontend Structure

## Objetivo

Documentar a estrutura frontend canônica do Backoffice Admin do HSC.

Este documento existe para registrar, de forma estável e auditável:

- a topologia lógica da SPA administrativa
- a organização por camadas e por domínio
- a estrutura recomendada de bootstrap, configuração e rotas
- a relação entre UI, sessão, guards e contratos backend
- a jornada real de autenticação administrativa publicada
- os princípios de evolução incremental da SPA

---

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Backoffice Admin](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Contexto administrativo imediato
- [Architecture Runtime](./backoffice-admin-architecture-runtime.md)
- [Admin API Contracts](./admin-api-contracts.md)
- [Auth, RBAC and Guards](./auth-rbac-and-guards.md)
- [Operational Runbooks](./backoffice-admin-operational-runbooks.md)

### Backend e runtime consumido
- [Auth API Operations](../04-infra-aws-lightsail/auth-api-operations.md)
- [Nginx Reverse Proxy](../04-infra-aws-lightsail/nginx-reverse-proxy.md)
- [Node Systemd](../04-infra-aws-lightsail/node-systemd.md)
- [Deploy / Release / Rollback](../04-infra-aws-lightsail/deploy-release-rollback.md)

### Governança e suporte
- [Documentation System](../00-governance/documentation-system.md)
- [References and Inventory](./backoffice-admin-references-inventory.md)

---

## Escopo

Este documento cobre:

- a estrutura lógica da SPA administrativa
- a abordagem Angular standalone-first
- a divisão entre `core`, `shared` e `features`
- a organização por domínio administrativo
- o shell de navegação do Backoffice
- o padrão de rotas iniciais
- o fluxo real de login admin por magic link
- o papel de `/auth/callback`
- o tratamento de loading, erro, vazio e acesso negado
- a postura recomendada para estado com Signals
- a convivência entre Signals e RxJS
- o papel do proxy local no desenvolvimento
- a diferença entre integração local e integração publicada

Este documento não cobre em profundidade:

- contrato endpoint a endpoint
- regras completas de RBAC
- detalhes completos de infraestrutura e deploy
- modelagem visual final do design system
- runbooks operacionais detalhados
- troubleshooting exaustivo de produção

Esses tópicos vivem em documentos adjacentes do contexto `05-backoffice-admin` e em documentos do contexto `04-infra-aws-lightsail`.

---

## Estado atual

O estado atual esperado do Backoffice Admin é:

- o Backoffice existe como SPA administrativa separada do Portal público
- a aplicação adota Angular standalone-first
- a organização prioriza MVP, legibilidade e baixo acoplamento
- a Auth API é a autoridade final de autenticação e autorização
- o frontend reflete sessão e permissão, não as inventa
- o login administrativo real publicado usa magic link
- a jornada publicada é `login -> request magic link -> callback -> dashboard`
- a SPA já opera com sessão real e cookies cross-subdomain
- o shell administrativo já é superfície real do produto
- `dashboard`, `seasons`, `news` e `events` continuam sendo os domínios iniciais

Regra importante:

- a SPA do Backoffice não deve nascer como coleção desordenada de telas CRUD
- ela deve nascer como produto administrativo com estrutura clara, previsível e escalável por domínio

---

## Source of truth

Este documento é canônico para:

- estrutura frontend do Backoffice
- topologia de diretórios e camadas da SPA
- superfícies reais de auth do lado frontend
- princípios de organização do admin shell
- relação entre sessão, guards e domínio

Documentos adjacentes:

- `admin-api-contracts.md`
- `auth-rbac-and-guards.md`
- `operational-runbooks.md`

Relações com outros contextos:

- a Auth API publicada e seus contratos operacionais vivem em `04-infra-aws-lightsail`
- este documento descreve como o frontend administrativo se organiza para consumir essa infraestrutura

---

## Princípios estruturais

A estrutura frontend do Backoffice deve obedecer aos seguintes princípios:

- standalone-first
- organização por domínio administrativo
- separação clara entre camadas transversais e features
- shell administrativo único e estável
- rotas protegidas por guards assíncronos
- backend como autoridade final de sessão e permissão
- baixo acoplamento entre features
- reaproveitamento controlado via `shared`
- infraestrutura transversal concentrada em `core`
- Signals como estratégia primária de estado local e derivado
- RxJS como ferramenta complementar para HTTP e interoperabilidade
- simplicidade antes de sofisticação
- clareza operacional antes de abstração excessiva

Princípio importante:

- qualquer abstração frontend que esconda demais o domínio deve ser evitada no MVP

---

## Stack e postura tecnológica

O Backoffice Admin adota explicitamente:

- Angular
- TypeScript
- componentes standalone
- bootstrap por `bootstrapApplication`
- providers globais em `app.config.ts`
- roteamento central em `app.routes.ts`
- Signals para estado local e derivado
- `computed()` para estado derivado
- `effect()` apenas para side effects reais
- RxJS para HTTP, streams externos e interoperabilidade
- shell administrativo com áreas autenticadas protegidas
- proxy local como ergonomia de desenvolvimento

O Backoffice Admin não adota, no MVP:

- arquitetura centrada em `NgModule`
- state management global pesado
- meta-framework interno de CRUD
- engine genérica de formulários
- abstrações prematuras de store

---

## Bootstrap e configuração da aplicação

A estrutura de entrada da aplicação segue o modelo moderno do Angular:

```text
src/
  main.ts

  app/
    app.component.ts
    app.component.html
    app.component.scss
    app.config.ts
    app.routes.ts
```

### `main.ts`

Responsável por:

* inicializar a aplicação
* executar `bootstrapApplication`
* bootstrapar o componente raiz standalone

### `app.config.ts`

Responsável por:

* registrar providers globais
* configurar Router
* configurar HttpClient
* configurar interceptors
* registrar providers transversais da aplicação

### `app.routes.ts`

Responsável por:

* centralizar a árvore principal de rotas
* declarar áreas públicas e protegidas
* delegar lazy loading de features quando necessário

### `app.component.*`

Responsável por:

* componente raiz standalone
* conter o `router-outlet` raiz
* iniciar a moldura mínima da aplicação

---

## Proxy local e integração publicada

No desenvolvimento local, a SPA usa proxy para conversar com a Auth API local.

Artefato esperado:

* `proxy.conf.json`

Objetivo:

* encaminhar chamadas a `/auth`, `/admin`, `/content` e `/health`
* reduzir fricção prematura com CORS e cookies no navegador
* permitir fluxo local previsível com sessão real

Regra importante:

* o proxy local é parte da ergonomia de desenvolvimento
* ele não é parte da arquitetura publicada

Em produção:

* o frontend usa a base publicada da Auth API
* a integração real ocorre com `https://auth-api.haxixesmokeclub.com`
* o Backoffice publicado é `https://backoffice.haxixesmokeclub.com`

Resumo:

* **local:** proxy
* **publicado:** base URL explícita da Auth API

---

## Topologia lógica recomendada

A estrutura lógica recomendada para a SPA é:

```text
src/
  main.ts

  app/
    app.component.ts
    app.component.html
    app.component.scss
    app.config.ts
    app.routes.ts

    core/
    shared/
    features/
      auth/
      dashboard/
      seasons/
      news/
      events/
      error-pages/
```

Leitura dessa topologia:

* `main.ts` define o bootstrap moderno
* `app.config.ts` centraliza providers globais
* `app.routes.ts` define a navegação principal
* `core` concentra infraestrutura transversal
* `shared` concentra reaproveitamento controlado
* `features` concentra os slices de domínio
* cada domínio administrativo deve ter fronteira própria

---

## Estado publicado da feature `/users`

A estrutura do Backoffice Admin publicado já não se limita mais aos domínios iniciais de dashboard e placeholders.

Estado efetivo publicado desta linha:

- a rota protegida `/users` já existe no shell administrativo
- o sidebar administrativo já expõe a entrada `Users`
- a feature já foi materializada em:
  - `src/app/features/users/data-access/users-admin-api.service.ts`
  - `src/app/features/users/pages/users-page/users-page.component.ts`
  - `src/app/features/users/pages/users-page/users-page.component.html`
  - `src/app/features/users/pages/users-page/users-page.component.scss`

Capacidades já validadas no ambiente publicado:

- listagem real de usuários
- criação de usuário
- alteração de role
- renomeação
- alteração de email

Leitura arquitetural correta neste estágio:

- `core/auth` continua transversal
- `users` já é um domínio funcional próprio dentro de `features/`
- a superfície `/users` depende da Auth API publicada para:
  - `GET /admin/users`
  - `POST /admin/users`
  - `PATCH /admin/users/:id`

Observação importante:

- a existência da feature `/users` no frontend não altera, por si só, o modelo efetivo de autorização publicado
- o shell administrativo publicado continua sendo, neste estágio, admin-only

---

## Estrutura detalhada recomendada

A estrutura inicial sugerida é:

```text
src/
  main.ts

  app/
    app.component.ts
    app.component.html
    app.component.scss
    app.config.ts
    app.routes.ts

    core/
      auth/
        auth-api.service.ts
        auth-session.store.ts
        models/
          auth.model.ts
          role.model.ts
      guards/
        auth.guard.ts
        admin-access.guard.ts
        role.guard.ts
      interceptors/
        auth.interceptor.ts
        auth-error.interceptor.ts
      layout/
        admin-shell/
        sidebar/
        header/
        page-container/
      config/
      services/
      models/

    shared/
      ui/
      table/
      forms/
      feedback/
      utils/
      types/

    features/
      auth/
        pages/
          login-page/
          auth-callback-page/
      dashboard/
        pages/
          dashboard-page/
      seasons/
        pages/
        components/
        services/
        store/
        models/
      news/
        pages/
        components/
        services/
        store/
        models/
      events/
        pages/
        components/
        services/
        store/
        models/
      error-pages/
        forbidden-page/
        not-found-page/
```

### `core/`

Camada de infraestrutura transversal da SPA.

Deve concentrar:

* auth administrativa
* estado de sessão
* guards
* interceptors
* layout principal do shell
* configuração global da aplicação

### `shared/`

Camada de reaproveitamento controlado.

Deve concentrar:

* componentes verdadeiramente genéricos
* helpers de UI
* blocos de feedback
* utilitários sem acoplamento de domínio

### `features/`

Camada de domínio administrativo.

Deve concentrar:

* páginas
* componentes específicos
* serviços por domínio
* stores por domínio
* models por domínio

Regra importante:

* regra de domínio não deve morar em `shared`
* regra de domínio não deve contaminar `core`

---

## Shell administrativo

O Backoffice nasce com um shell administrativo explícito.

O shell é a moldura estável da aplicação e deve conter:

* sidebar de navegação
* header simples
* área principal de conteúdo
* estado de carregamento inicial
* área de feedback global
* tratamento de acesso negado
* tratamento de erro de navegação

O shell deve existir antes dos domínios administrativos completos.

Objetivo:

* padronizar a experiência de administração
* evitar que cada tela resolva layout por conta própria
* sustentar evolução incremental de features

### Estrutura sugerida do layout

```text
core/layout/
  admin-shell/
  sidebar/
  header/
  page-container/
```

Responsabilidades esperadas:

#### `admin-shell/`

* componente raiz do layout autenticado
* estrutura fixa da área administrativa
* encaixe do `router-outlet` protegido

#### `sidebar/`

* navegação por domínio
* destaque de rota ativa
* ocultação de entradas não autorizadas, quando aplicável

#### `header/`

* contexto de tela
* identidade do operador autenticado, quando disponível
* status da sessão
* papel atual do operador

#### `page-container/`

* consistência de espaçamento e composição visual
* título, subtítulo e área principal da página
* uso padronizado em listagens e formulários

---

## Rotas iniciais recomendadas

A navegação inicial da SPA deve ser simples, explícita e coerente com o runtime publicado.

Rotas reais ou esperadas:

```text
/login
/auth/callback
/dashboard
/403
/404
/seasons
/news
/events
```

Evolução natural por domínio:

```text
/seasons/new
/seasons/:slug/edit

/news/new
/news/:id/edit

/events/new
/events/:id/edit
```

Leitura dessas rotas:

* `/login` representa o ponto de entrada administrativo
* `/auth/callback` representa a superfície real do consumo do magic link do ponto de vista da SPA
* `/dashboard` representa a área autenticada inicial
* cada domínio terá sua rota de listagem e suas rotas de formulário
* `/403` e `/404` existem como superfícies reais do produto

Regra importante:

* não criar profundidade de rotas desnecessária no MVP
* começar com superfícies claras e previsíveis

---

## Jornada canônica de autenticação administrativa

A jornada administrativa real publicada é:

1. operador acessa `/login`
2. informa email administrativo
3. frontend chama `POST /auth/magic-link/request`
4. usuário recebe email com o magic link
5. link aponta para `GET /auth/magic-link/consume?token=...`
6. backend consome o token, cria a sessão e redireciona para `/auth/callback?status=ok`
7. a SPA resolve a sessão
8. navega para `/dashboard`

Em caso de falha no callback, a SPA precisa tratar códigos como:

* `invalid_or_expired_link`
* `consume_failed`
* `missing_token`
* `forbidden`

Regra importante:

* a autoridade da sessão está no backend
* a SPA deve reagir ao estado real, não adivinhar sucesso de login

---

## Estratégia de proteção de rotas

A proteção de rotas deve seguir uma hierarquia simples:

1. resolução assíncrona da sessão
2. validação de acesso administrativo
3. validação de permissão por rota, quando necessário

A estrutura de guards deve morar em:

```text
core/guards/
```

### `auth.guard`

* aguardar resolução da sessão
* permitir área protegida apenas com sessão válida
* redirecionar para `/login` quando não autenticado

### `admin-access.guard`

* distinguir `unauthenticated` de `forbidden`
* encaminhar para `/403` quando houver autenticação sem acesso suficiente

### `role.guard`

* proteger ações ou rotas mais sensíveis por papel
* ser introduzido conforme os domínios exigirem

Regra importante:

* guards do admin devem ser assíncronos quando dependem de sessão real
* `unknown` não deve ser tratado como “sem sessão” antes da resolução

---

## Auth, sessão e contratos backend

A SPA administrativa já conversa com contrato real de sessão.

Superfícies centrais:

* `GET /auth/session`
* `POST /auth/magic-link/request`
* `GET /auth/magic-link/consume`

Papel no frontend:

* bootstrap do estado auth
* sustentação dos guards
* persistência de sessão após refresh
* jornada real de login admin
* base para UI por papel

Superfície auxiliar de desenvolvimento:

* `POST /auth/dev/bootstrap-session`

Papel no frontend local:

* criar sessão local controlada
* destravar desenvolvimento do admin
* não representar login final de produção

---

## Sessão real, cookies e integração cross-subdomain

A sessão administrativa publicada é baseada em cookie emitido pela Auth API.

Implicações para o frontend:

* chamadas de sessão precisam usar credenciais
* a SPA depende do backend para saber se a sessão é válida
* a transição `consume -> callback -> /auth/session` precisa ser tratada como fluxo real de rede
* o estado de auth deve sobreviver a refresh

No ambiente publicado:

* `backoffice.haxixesmokeclub.com` consome `auth-api.haxixesmokeclub.com`
* a sessão é cookie-based
* a SPA precisa usar `withCredentials` nas chamadas relevantes de auth

Regra importante:

* cookie, sessão e autorização são responsabilidade do backend
* o frontend não deve implementar “atalhos” paralelos de auth

---

## Estratégia de estado com Signals

A estratégia primária de estado local e derivado do Backoffice é baseada em Signals.

Aplicações recomendadas:

* estado da sessão administrativa
* flags de loading
* derived state de permissão
* estado de página
* estado de formulários locais
* estado leve por feature

### Uso recomendado

* `signal()` para estado mutável local
* `computed()` para estado derivado
* `effect()` apenas quando houver side effect real
* RxJS para HTTP e integração externa

Regra importante:

* a store de sessão é transversal
* stores de domínio devem existir por feature quando necessário
* não introduzir state management global extra cedo demais

---

## Store de sessão e reload explícito

A store de sessão do admin é uma peça central do runtime.

Ela deve sustentar, no mínimo:

* estado `unknown`
* estado `authenticated`
* estado `unauthenticated`
* estado de erro quando aplicável
* dados do operador autenticado
* papel efetivo resolvido pelo backend

A store depende de:

* `GET /auth/session`
* `reloadSession()`

Papel de `reloadSession()`:

* resolver sessão após refresh
* resolver sessão após callback
* sustentar guards e header
* permitir rechecagem explícita em telas técnicas quando necessário

Regra importante:

* o frontend não deve assumir autenticação só porque o callback retornou `status=ok`
* ele deve sempre confirmar a sessão no backend

---

## Callback com retry curto

A superfície `/auth/callback` existe para fechar a jornada real do magic link.

Responsabilidades dessa página:

* ler `status` ou `error` da query string
* tentar resolver a sessão
* em caso de `status=ok`, reconsultar o backend
* tolerar um pequeno race condition entre emissão de cookie e primeira consulta de sessão
* navegar para `/dashboard` quando a sessão for validada
* mostrar mensagem clara e botão de retorno ao login quando a validação falhar

Regra importante:

* o callback não é uma tela “decorativa”
* ele é uma parte real da infraestrutura de login do admin

---

## Experiência local de auth

A UX local do Backoffice pode admitir:

* exibição de status da sessão na `/login`
* botão para criar sessão local de desenvolvimento
* botão para resolver sessão atual
* dashboard técnico com recarga explícita de sessão
* header com `status`, `role` e identidade do operador

Esses elementos não são ruído.

Eles são parte da ergonomia saudável da linha session-first durante desenvolvimento local.

---

## Organização por domínio

A aplicação deve crescer por domínio administrativo real.

Domínios iniciais do contexto:

* `dashboard`
* `seasons`
* `news`
* `events`

Leitura prática:

* não é necessário implementar todos no primeiro corte
* o shell e o fluxo de auth já validam a espinha dorsal
* `seasons` continua sendo o melhor primeiro domínio forte
* `news` tende a entrar logo depois
* `events` ainda exige maturação contratual adicional

---

## Estrutura mínima por feature

A estrutura mínima por feature pode ser lida assim:

```text
features/<dominio>/
  pages/
  components/
  services/
  store/
  models/
```

### `pages/`

* superfícies de rota
* composição principal da feature

### `components/`

* blocos específicos do domínio
* não genéricos por padrão

### `services/`

* integração HTTP do domínio
* adaptação de payload
* helpers de API específicos do slice

### `store/`

* estado local do domínio
* filtros, seleção, carregamento e derived state

### `models/`

* tipos e contratos específicos do domínio
* shapes de leitura e escrita

---

## Artefatos que devem nascer cedo

Os artefatos que mais cedo devem existir na implementação são:

### Estruturais

* `main.ts`
* `app.config.ts`
* `app.routes.ts`
* shell administrativo

### Acesso

* `auth-session.store.ts`
* `auth-api.service.ts`
* guards
* interceptors básicos
* `login-page`
* `auth-callback-page`

### Primeiro domínio

* `features/seasons/`
* listagem
* formulário
* ações de lifecycle

Motivo:

* esses artefatos validam a espinha dorsal do Backoffice

---

## Artefatos que podem esperar um pouco mais

Os artefatos que podem nascer em fase seguinte são:

* componentes shared mais sofisticados
* dashboard mais rico
* refinamentos avançados de tabela
* filtros e paginação avançados
* telemetria frontend mais elaborada
* evolução de `events` além do CRUD básico
* abstrações adicionais de store ou UI

Motivo:

* não são necessários para validar a estrutura canônica do admin

---

## Regras de fronteira entre documentos

Este documento não substitui:

* `admin-api-contracts.md`
* `auth-rbac-and-guards.md`
* `operational-runbooks.md`

Leitura correta:

* este documento define **estrutura**
* `admin-api-contracts.md` define **contratos**
* `auth-rbac-and-guards.md` define **semântica de auth e acesso**
* `operational-runbooks.md` define **procedimentos repetíveis**

---

## Limites atuais

Este documento assume como verdade operacional:

* existência de sessão real publicada
* fluxo de magic link funcional
* callback administrativo funcional
* dashboard acessível após autenticação
* Auth API como backend dinâmico central do admin

Este documento não afirma, por si só:

* que todos os domínios administrativos já estão completos
* que o design system está estabilizado
* que todos os guards por papel já estão implementados
* que toda a observabilidade frontend já existe

---

## Critério de pronto

Este documento está suficientemente pronto quando:

* a estrutura `core/shared/features` estiver refletida no repo
* o shell administrativo estiver materializado
* `/login` e `/auth/callback` existirem como superfícies reais
* a store de sessão sustentar guards e callback
* a SPA publicada consumir corretamente a Auth API publicada
* os domínios administrativos puderem crescer sem colapsar a organização da app

---

## Última revisão

* Revisado após a materialização do fluxo real de magic link admin publicado
* Revisado após a consolidação do callback `/auth/callback`
* Revisado após a estabilização da sessão cookie-based cross-subdomain
* Revisado após a consolidação da Auth API como autoridade final de sessão e permissão