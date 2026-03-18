# Frontend Structure

## Objetivo

Documentar a estrutura frontend canônica do Backoffice Admin do HSC, assumindo explicitamente a stack:

- Angular 20
- TypeScript
- standalone-first
- Angular Router
- HttpClient via providers
- Signals como estratégia primária de estado local e derivado
- RxJS como ferramenta complementar para fluxos assíncronos, integração HTTP e interoperabilidade

Este documento existe para registrar, de forma estável e auditável:

- a topologia lógica da SPA administrativa
- a organização por camadas e por domínio
- a estrutura recomendada de bootstrap, configuração e rotas
- os padrões mínimos para páginas administrativas
- a relação entre UI, sessão, RBAC e contratos backend
- os princípios de evolução incremental da SPA

---

## Escopo

Este documento cobre:

- a estrutura lógica da SPA administrativa
- a abordagem Angular 20 standalone-first
- a divisão entre `core`, `shared` e `features`
- a organização por domínio administrativo
- o shell de navegação do Backoffice
- o padrão de rotas iniciais
- o padrão de listagens, formulários e ações
- o tratamento de loading, erro, vazio e acesso negado
- a postura recomendada para estado com Signals
- a convivência entre Signals e RxJS

Este documento não cobre em profundidade:

- contrato detalhado endpoint a endpoint
- regras completas de RBAC
- runbooks operacionais
- detalhes visuais finais de design system
- infraestrutura de deploy
- Nginx, hostnames ou build pipeline
- persistência backend

Esses tópicos vivem em outros documentos do contexto ou em contextos canônicos adjacentes.

---

## Estado atual

O estado esperado da estrutura frontend, nesta fase, é:

- o Backoffice nasce como SPA administrativa separada do Portal público
- a stack alvo do frontend é Angular 20 + TypeScript + Signals
- a aplicação deve nascer em modo standalone-first
- a estrutura deve priorizar MVP, legibilidade e baixo acoplamento
- a organização precisa refletir domínios administrativos reais
- a Auth API é a autoridade final de autenticação e autorização
- o frontend deve refletir permissão e estado, não inventá-los
- `seasons`, `news` e `events` são os domínios iniciais
- o shell administrativo deve nascer primeiro
- o front deve evitar overengineering de state management no início

Regra importante:

- a SPA do Backoffice não deve nascer como coleção desordenada de telas CRUD
- ela deve nascer como produto administrativo com estrutura clara, previsível e escalável por domínio

---

## Princípios estruturais

A estrutura frontend do Backoffice deve obedecer aos seguintes princípios:

- standalone-first
- organização por domínio administrativo
- separação clara entre camadas transversais e features
- rotas protegidas por guard
- shell administrativo único e estável
- baixo acoplamento entre features
- contratos explícitos com o backend
- reaproveitamento controlado via `shared`
- lógica transversal concentrada em `core`
- Signals como estratégia primária de estado local e derivado
- RxJS como complemento, não como centro arquitetural da UI
- simplicidade antes de sofisticação
- clareza operacional antes de abstração excessiva

Princípio importante:

- qualquer abstração frontend que esconda demais o domínio deve ser evitada no MVP

---

## Decisões tecnológicas do contexto

O Backoffice Admin adota explicitamente:

- Angular 20
- TypeScript
- componentes standalone
- bootstrap por `bootstrapApplication`
- providers globais em `app.config.ts`
- roteamento central em `app.routes.ts`
- Signals para estado local e derivado
- `computed()` para valores derivados
- `effect()` apenas para side effects reais
- RxJS para HTTP, streams externos e interoperabilidade

O Backoffice Admin não adota, no MVP:

- arquitetura centrada em `NgModule`
- biblioteca global de state management
- meta-framework interno de CRUD
- engine genérica de formulários
- abstrações excessivas de store

---

## Bootstrap e configuração da aplicação

A estrutura de entrada da aplicação deve seguir o modelo moderno do Angular:

```text
src/
  main.ts
  app/
    app.component.ts
    app.config.ts
    app.routes.ts
```

### `main.ts`

Responsável por:

- inicializar a aplicação
- executar `bootstrapApplication`
- bootstrapar o componente raiz standalone

### `app.config.ts`

Responsável por:

- registrar providers globais
- configurar Router
- configurar HttpClient
- configurar interceptors
- registrar providers transversais da aplicação

### `app.routes.ts`

Responsável por:

- centralizar a árvore principal de rotas
- declarar áreas protegidas
- delegar lazy loading de features quando necessário

### `app.component.ts`

Responsável por:

- componente raiz standalone
- conter o `router-outlet` raiz
- iniciar a moldura mínima da aplicação

---

## Topologia lógica recomendada

A estrutura lógica recomendada para a SPA é:

```text
src/
  main.ts
  app/
    app.component.ts
    app.config.ts
    app.routes.ts

    core/
    shared/
    features/
      dashboard/
      seasons/
      news/
      events/
```

Leitura dessa topologia:

- `main.ts` define o bootstrap moderno
- `app.config.ts` centraliza providers globais
- `app.routes.ts` define a navegação principal
- `core` concentra infraestrutura transversal da aplicação
- `shared` concentra componentes e utilidades reaproveitáveis
- `features` concentra os slices de domínio
- cada domínio administrativo deve ter fronteira própria

---

## Estrutura detalhada recomendada

A estrutura inicial sugerida é:

```text
src/
  main.ts

  app/
    app.component.ts
    app.config.ts
    app.routes.ts

    core/
      auth/
      guards/
      interceptors/
      layout/
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
      dashboard/
        pages/
        components/
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
```

Essa estrutura deve ser lida assim:

### `core/`

Camada de infraestrutura transversal da SPA.

Deve concentrar:

- autenticação
- sessão do usuário atual
- guards
- interceptors HTTP
- configuração global
- serviços transversais
- layout administrativo
- modelos compartilhados estritamente centrais

O que entra em `core`:

- tudo que a aplicação inteira depende para funcionar
- tudo que não pertence a um único domínio de negócio
- tudo que governa navegação, acesso e integração transversal

O que não deve entrar em `core`:

- lógica específica de `seasons`
- lógica específica de `news`
- lógica específica de `events`
- componentes de tela de feature
- stores específicas de domínio

---

### `shared/`

Camada de reaproveitamento controlado.

Deve concentrar:

- componentes visuais simples e reutilizáveis
- blocos genéricos de tabela
- blocos genéricos de formulário
- feedback visual
- estados vazios
- loaders
- helpers utilitários
- types compartilhados realmente neutros

O objetivo de `shared` não é virar um “cemitério de tudo que sobrou”.

Regra importante:

- só deve ir para `shared` aquilo que é realmente transversal e reutilizável sem contaminar o domínio

---

### `features/`

Camada de domínios administrativos.

Cada domínio deve ter seu próprio slice.

Os slices iniciais são:

- `dashboard`
- `seasons`
- `news`
- `events`

Cada feature deve conter, no mínimo:

- `pages/`
- `components/`
- `services/`
- `store/`
- `models/`

Leitura dessa convenção:

- `pages/` contém superfícies de rota
- `components/` contém blocos locais da feature
- `services/` contém integração e orquestração da feature
- `store/` contém estado local com Signals
- `models/` contém tipos e mapeamentos próprios do domínio

---

## Abordagem Angular 20: standalone-first

O Backoffice deve nascer explicitamente em modo standalone-first.

Isso significa:

- componentes de página standalone
- componentes compartilhados standalone
- sem dependência estrutural de `NgModule` para código novo
- imports declarados por componente ou por rota
- providers globais centralizados em `app.config.ts`
- lazy loading preferencial por rotas/features

Regra importante:

- se algum legado exigir integração com módulos antigos, isso deve ser tratado como adaptação pontual
- o desenho canônico do Backoffice não deve ser module-first

---

## Shell administrativo

O Backoffice deve nascer com um shell administrativo explícito.

O shell é a moldura estável da aplicação e deve conter:

- sidebar de navegação
- header simples
- área principal de conteúdo
- estado de carregamento inicial
- área de feedback global
- tratamento de acesso negado
- tratamento de erro de navegação

O shell deve existir antes dos domínios administrativos completos.

Objetivo:

- padronizar a experiência de administração
- evitar que cada tela resolva layout por conta própria
- sustentar evolução incremental de features

---

## Estrutura sugerida do layout

A estrutura lógica do layout pode ser lida assim:

```text
core/layout/
  admin-shell/
  sidebar/
  header/
  page-container/
```

Responsabilidades esperadas:

### `admin-shell/`

- componente raiz do layout autenticado
- estrutura fixa da área administrativa
- encaixe do `router-outlet` protegido

### `sidebar/`

- navegação por domínio
- destaque de rota ativa
- ocultação de entradas não autorizadas, quando aplicável

### `header/`

- contexto de tela
- identidade do operador autenticado, se disponível
- ações globais mínimas

### `page-container/`

- consistência de espaçamento e composição visual
- título, subtítulo e área principal da página
- uso padronizado em listagens e formulários

---

## Rotas iniciais recomendadas

A navegação inicial da SPA deve ser simples e explícita.

Rotas sugeridas:

```text
/login
/dashboard

/seasons
/seasons/new
/seasons/:slug/edit

/news
/news/new
/news/:id/edit

/events
/events/new
/events/:id/edit

/403
/404
```

Leitura dessas rotas:

- `/login` representa o ponto de entrada administrativo
- `/dashboard` representa o shell técnico inicial
- cada domínio tem sua rota de listagem e suas rotas de formulário
- `/403` e `/404` devem existir como superfícies reais do produto

Regra importante:

- não criar profundidade de rotas desnecessária no MVP
- começar com superfícies claras e previsíveis

---

## Estratégia de proteção de rotas

A proteção de rotas deve seguir uma hierarquia simples:

1. validação de sessão
2. validação de acesso administrativo
3. validação de permissão por rota, quando necessário

A estrutura de guardas deve morar em:

```text
core/guards/
```

Exemplos de responsabilidades:

- guard de autenticação
- guard de acesso administrativo
- guard de papel mínimo por área sensível

Princípio importante:

- o guard controla entrada na rota
- a Auth API continua controlando autorização real da mutação

---

## Padrão de páginas por domínio

Cada domínio administrativo deve nascer com dois tipos centrais de página:

### Página de listagem

Responsável por:

- listar entidades
- exibir loading
- exibir erro
- exibir empty state
- oferecer ação principal de criação
- oferecer ações por item

### Página de formulário

Responsável por:

- criação
- edição
- carregamento do dado inicial quando aplicável
- validação básica
- submit
- feedback de sucesso ou erro
- navegação de retorno

Regra importante:

- create e edit podem compartilhar a mesma página/componente de formulário
- isso reduz duplicação e acelera o MVP

---

## Estrutura de estado com Signals

A estratégia primária de estado do Backoffice deve ser baseada em Signals.

Cada domínio deve concentrar seu estado em uma store local da feature, por exemplo:

```text
features/seasons/
  services/
    seasons-api.service.ts
  store/
    seasons.store.ts
```

Responsabilidades esperadas da store da feature:

- manter estado de listagem
- manter estado de detalhe/edição quando necessário
- expor loading flags
- expor erros de tela
- expor filtros simples
- expor dados derivados com `computed`
- orquestrar chamadas ao serviço HTTP do domínio

A store da feature não deve:

- virar mini-framework genérico
- centralizar lógica de outros domínios
- esconder invariantes importantes do domínio

---

## Regras de uso de Signals

### Use `signal()` para estado mutável local

Exemplos naturais:

- lista carregada
- item selecionado
- loading
- submitting
- filtro atual
- mensagem local de erro

### Use `computed()` para estado derivado

Exemplos naturais:

- lista filtrada
- total de itens visíveis
- flags derivadas do estado atual
- permissões derivadas já resolvidas na camada de tela
- título de página derivado do modo create/edit

### Use `effect()` apenas para side effects reais

Exemplos aceitáveis:

- persistir algo em storage local
- logging
- sincronização com API imperativa
- integração com componente externo não reativo
- telemetria

### Não use `effect()` para propagação de estado

Evitar:

- copiar signal para outro signal sem necessidade
- encadear mutações de estado derivado
- substituir `computed()` por `effect()`
- criar fluxo de negócio centrado em side effects

Regra importante:

- estado derivado deve nascer de `computed`
- `effect` deve ser a exceção, não a regra

---

## Convivência entre Signals e RxJS

Signals e RxJS não competem; eles cumprem papéis diferentes.

No Backoffice HSC:

### Signals

Devem governar:

- estado de tela
- estado local por feature
- derivação de estado
- feedback visual
- composição da UI

### RxJS

Deve governar:

- chamadas HTTP
- streams assíncronos
- composição com APIs observáveis
- integração com libs ou fluxos externos
- cenários onde Observable é a forma natural da fonte

Regra de convivência:

- HTTP continua natural em serviços Angular
- a feature pode consumir o resultado do serviço e consolidar o estado em Signals
- a UI não precisa virar uma árvore inteira baseada em `async` pipe para tudo

---

## Padrão de composição por feature

A estrutura recomendada de uma feature deve seguir algo nessa linha:

```text
features/seasons/
  pages/
    seasons-list-page/
    season-form-page/
  components/
    season-form/
    season-actions/
    season-status-badge/
  services/
    seasons-api.service.ts
  store/
    seasons.store.ts
  models/
    season.model.ts
    season-form.model.ts
```

Leitura recomendada:

### `pages/`

Superfícies roteáveis da feature.

Devem ser leves e orquestrar:

- carregamento da página
- integração com shell e container
- delegação para componentes locais
- ligação entre rota e store da feature

### `components/`

Blocos internos da feature.

Devem conter:

- formulário do domínio
- ações contextuais
- badges ou blocos específicos do domínio
- partes reaproveitáveis apenas dentro daquela feature

### `services/`

Responsáveis por:

- integração HTTP
- chamadas à Auth API
- mapeamentos de payload quando necessário

### `store/`

Responsável por:

- consolidar estado da feature
- expor signals e computed signals
- orquestrar ciclo de loading/erro/sucesso
- manter a lógica de tela perto do domínio

### `models/`

Deve conter:

- tipos de leitura
- tipos de formulário
- mapeamentos simples
- enums específicos do domínio

---

## Estratégia de integração HTTP

A integração com a Auth API deve seguir estes princípios:

- uma feature não deve chamar HTTP diretamente em múltiplos componentes dispersos
- chamadas devem ser centralizadas em serviços da feature
- mapeamento entre payload e modelo de tela deve ser explícito
- tratamento de erro deve ser consistente

Estrutura sugerida:

- `app.config.ts` para `provideHttpClient(...)`
- `core/interceptors/` para comportamento transversal
- `features/<dominio>/services/` para endpoints e fluxos do domínio

O que deve ficar em interceptor:

- headers globais
- tratamento base de erro
- reação transversal a 401, quando apropriado

O que não deve ficar em interceptor:

- regras específicas de `seasons`
- regras específicas de `news`
- regras específicas de `events`

---

## Tratamento de 401, 403 e erros funcionais

O Backoffice deve tratar esses estados como parte do produto.

### 401

Deve significar:

- sessão ausente
- sessão expirada
- autenticação inválida

Comportamento esperado:

- interromper fluxo protegido
- redirecionar de forma consistente
- informar o operador de forma clara, quando aplicável

### 403

Deve significar:

- usuário autenticado sem permissão suficiente

Comportamento esperado:

- mostrar tela ou estado explícito de acesso negado
- não mascarar esse estado como erro genérico

### Erros de domínio

Exemplos:

- season já ativa
- slug inválido
- publicação não permitida
- recurso não encontrado para edição

Comportamento esperado:

- mensagem útil
- feedback localizado ou contextual
- sem tratar tudo como “erro inesperado”

---

## Padrão de feedback visual

O Backoffice deve oferecer uma linguagem consistente de feedback.

Estados mínimos por página:

- loading inicial
- erro de carregamento
- empty state
- sucesso de mutação
- erro de mutação
- ação em progresso

Componentes reutilizáveis recomendados em `shared/feedback/`:

- loader de página
- bloco de erro
- estado vazio
- confirmação de ação destrutiva
- mensagem de sucesso simples

Regra importante:

- ações destrutivas não devem usar feedback ambíguo
- ações sensíveis não devem depender de atualização otimista no MVP

---

## Padrão de tabelas

As listagens administrativas devem seguir um padrão estável.

Características mínimas:

- coluna principal identificadora
- colunas de estado relevantes
- ação primária de criar
- ações por linha
- loading
- erro
- vazio

No MVP, evitar:

- grid excessivamente sofisticado
- filtros complexos demais
- customização profunda por usuário
- paginação avançada antes da necessidade real

A tabela deve servir ao domínio, não dominar a arquitetura.

---

## Padrão de formulários

Os formulários administrativos devem ser pragmáticos.

Características mínimas:

- modelo de formulário explícito
- validação básica local
- mensagens claras
- submit bloqueado durante envio
- botão de salvar
- navegação de retorno
- hidratação inicial no modo edição

Ações destrutivas devem ficar fora do corpo principal do formulário quando possível.

Regra importante:

- o formulário deve refletir o domínio
- ele não deve virar componente genérico abstrato cedo demais

---

## Organização inicial por domínio

### Dashboard

Objetivo:

- servir como entrada autenticada inicial
- validar shell, sessão e navegação
- expor estado técnico mínimo do admin

Não precisa nascer como dashboard analítico sofisticado.

---

### Seasons

Deve ser o primeiro domínio relevante.

Motivos:

- invariantes mais claras
- lifecycle explícito
- ótima feature para validar listagem, formulário e ações sensíveis

Estrutura mínima esperada:

- listagem
- create/edit
- activate
- close

---

### News

Deve vir logo depois de `seasons`.

Motivos:

- superfície administrativa já mais madura no ecossistema
- bom domínio para validar workflow editorial mínimo

Estrutura mínima esperada:

- listagem
- create/edit
- publish
- unpublish
- delete

---

### Events

Deve vir depois de `news`.

Motivos:

- ainda pede reconciliação de produto mais aberta
- especialmente na separação entre gestão administrativa e interação pública

Estrutura mínima esperada:

- listagem
- create/edit
- delete ou cancelamento

---

## Convenções de nomenclatura

A estrutura deve seguir nomes previsíveis.

Recomendação:

- pasta por domínio em minúsculas
- componentes com nomes explícitos
- páginas com sufixo `page`
- stores com sufixo `store`
- serviços de integração com sufixo `api.service`
- evitar nomes genéricos como `manager`, `handler`, `helper` sem contexto

Exemplos bons:

- `season-form-page`
- `seasons-list-page`
- `seasons-api.service.ts`
- `seasons.store.ts`
- `season-status-badge`

Exemplos ruins:

- `crud-page`
- `general-form`
- `admin-helper`
- `manager-service`

---

## Limites de abstração no MVP

Para o MVP, evitar:

- meta-framework interno de CRUD
- fábrica genérica de telas administrativas
- engine genérica de formulários baseada só em schema
- store global pesada
- design system complexo demais
- excesso de wrappers abstratos

Motivo:

- isso aumenta custo cognitivo cedo demais
- esconde o domínio
- dificulta troubleshooting
- atrasa entrega real

O Backoffice deve crescer a partir de repetição observada, não de abstração antecipada.

---

## Estrutura mínima para o primeiro release funcional

A estrutura mínima do primeiro release funcional pode ser lida assim:

```text
src/
  main.ts
  app/
    app.component.ts
    app.config.ts
    app.routes.ts

    core/
      auth/
      guards/
      interceptors/
      layout/

    shared/
      feedback/
      ui/

    features/
      dashboard/
      seasons/
```

Leitura prática:

- não é necessário implementar todos os domínios no primeiro corte
- o shell e `seasons` já podem validar quase toda a espinha dorsal da SPA
- `news` e `events` entram em seguida sem exigir reescrita estrutural

---

## Riscos estruturais a evitar

Os principais riscos desta camada são:

### Risco 1 — misturar tudo em `shared`

Efeito:
- perda de fronteira de domínio
- reutilização ilusória
- acoplamento escondido

### Risco 2 — colocar regra de domínio em `core`

Efeito:
- camada transversal contaminada
- dificuldade de manutenção
- erosão da estrutura

### Risco 3 — abstrair CRUD cedo demais

Efeito:
- código difícil de entender
- telas parecidas por fora, frágeis por dentro
- dificuldade de adaptar invariantes reais de cada domínio

### Risco 4 — depender do frontend para autorização

Efeito:
- falsa sensação de segurança
- inconsistência entre UI e backend

### Risco 5 — acoplar Backoffice ao Portal

Efeito:
- erosão da separação de camadas
- confusão entre produto público e produto administrativo

### Risco 6 — usar `effect()` como motor principal de estado

Efeito:
- loops desnecessários
- propagação opaca de estado
- troubleshooting mais difícil
- distorção da modelagem correta com `computed()`

---

## Ordem de implementação recomendada

A ordem recomendada para materializar esta estrutura é:

1. `main.ts`, `app.config.ts` e `app.routes.ts`
2. `core/layout`
3. `core/auth`, `core/guards` e `core/interceptors`
4. `features/dashboard`
5. `features/seasons`
6. `shared/feedback`, `shared/forms`, `shared/table`
7. `features/news`
8. `features/events`

Essa ordem valida:

- bootstrap moderno
- shell
- acesso
- primeiro domínio forte
- reutilização real
- expansão controlada

---

## Resumo executivo

A estrutura frontend do Backoffice Admin do HSC deve nascer como SPA administrativa em Angular 20, organizada por domínio, com bootstrap moderno, componentes standalone, shell único e stores locais por feature baseadas em Signals.

A regra central é:

- `main.ts` para bootstrap
- `app.config.ts` para providers globais
- `app.routes.ts` para navegação
- `core` para infraestrutura transversal
- `shared` para reaproveitamento real
- `features` para domínios administrativos
- `store/` por domínio para estado local com Signals

O MVP deve começar simples, por slices claros, evitando abstração excessiva e preservando a separação entre Backoffice, Portal e backend dinâmico.