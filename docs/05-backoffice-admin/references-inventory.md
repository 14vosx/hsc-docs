# References and Inventory

## Objetivo

Consolidar o inventário de referências, evidências, artefatos, componentes, paths e pendências do contexto `05-backoffice-admin`.

Este documento existe para registrar, de forma estável e auditável:

- quais documentos alimentam este contexto
- quais artefatos reais do Backoffice já existem
- quais contratos administrativos já são reais
- quais componentes e paths estruturam a SPA administrativa
- quais dependências externas sustentam o Backoffice
- quais itens ainda estão pendentes de reconciliação
- quais são os limites documentais do contexto

---

## Escopo

Este documento cobre:

- documentos de origem do contexto
- artefatos reais do Backoffice Admin
- componentes estruturais do frontend
- contratos administrativos relevantes já materializados
- fluxo local de autenticação administrativa
- paths e arquivos críticos da SPA
- dependências externas do contexto
- comandos úteis de validação local
- pendências ainda abertas
- limites documentais do contexto

Este documento não cobre em profundidade:

- detalhes completos de runtime da Auth API
- deploy final do frontend administrativo
- hostnames definitivos de publicação do Backoffice
- runbooks completos de domínio
- modelagem detalhada de `news`, `seasons` e `events`
- detalhes completos de banco e Nginx
- histórico completo de mudanças

Esses tópicos vivem em documentos especializados dos contextos `04` e `05`, além de impl-logs e material legado.

---

## Estado atual

O contexto `05-backoffice-admin` já possui base real suficiente para ser tratado como contexto administrativo vivo do ecossistema HSC.

Neste estágio, já existe confirmação suficiente para fixar sem ambiguidade:

- aplicação administrativa própria em Angular 21
- arquitetura standalone-first
- uso de TypeScript
- uso de Signals para estado de sessão
- shell administrativo funcional
- header, sidebar e page container materializados
- `/login`, `/dashboard`, `/403` e `/404`
- guards assíncronos protegendo a área administrativa
- store de sessão administrativa já conectada a contrato real
- integração real com `GET /auth/session`
- bootstrap local controlado via `POST /auth/dev/bootstrap-session`
- proxy local para a Auth API durante desenvolvimento
- fluxo local validado:
  - criar sessão local
  - entrar no dashboard
  - dar refresh em rota protegida
  - manter sessão válida

Também já existe clareza suficiente para registrar:

- o Backoffice continua separado do Portal público
- a Auth API continua sendo a autoridade final de autenticação e autorização
- o fluxo administrativo normal do produto é session-first
- `x-admin-key` continua sendo contingência backend, não jornada normal do frontend
- `seasons` continua sendo o primeiro domínio forte recomendado para evolução do admin

---

## Source of truth / evidências

As evidências que sustentam este contexto se dividem em cinco grupos principais.

### 1. Documentação canônica do Backoffice

Usada para:

- definir fronteira do contexto
- registrar arquitetura administrativa
- sustentar auth, guards, contratos e runbooks
- posicionar o Backoffice dentro da estrutura HSC

### 2. Documentação canônica da Auth API

Usada para:

- confirmar contratos administrativos reais
- confirmar o modelo session-first
- confirmar `GET /auth/session`
- confirmar a existência e o papel do bootstrap local dev-only
- confirmar a Auth API como autoridade final

### 3. Implementação real do Backoffice local

Usada para:

- confirmar stack efetiva em Angular 21
- confirmar estrutura de `core/`, `features/` e layout
- confirmar store de sessão com Signals
- confirmar guards assíncronos
- confirmar proxy local e UX de auth local

### 4. Implementação real local da Auth API

Usada para:

- confirmar `GET /auth/session`
- confirmar `POST /auth/dev/bootstrap-session`
- confirmar tabela `sessions`
- confirmar `users.role`
- confirmar fallback administrativo por `x-admin-key`

### 5. Fluxo local validado ponta a ponta

Usado para:

- confirmar subida da Auth API local
- confirmar subida do Backoffice local
- confirmar bootstrap de sessão
- confirmar `/dashboard` autenticado
- confirmar refresh em rota protegida
- confirmar recarga explícita de sessão

Regra principal:

- quando houver conflito entre expectativa documental e implementação validada, prevalece a implementação validada até reconciliação explícita

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/05-backoffice-admin/README.md`
- `docs/05-backoffice-admin/architecture-runtime.md`
- `docs/05-backoffice-admin/frontend-structure.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/operational-runbooks.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`

Este documento não substitui nenhum dos arquivos acima.  
Ele funciona como fechamento de inventário e referência do contexto administrativo.

---

## Documentos de origem

Os documentos abaixo são as principais fontes de extração e reconciliação deste contexto.

### Fontes primárias

- documentação canônica do contexto `05-backoffice-admin`
- documentação canônica da Auth API em `04-infra-aws-lightsail`
- índice mestre e documentos de governança

### Fontes secundárias

- implementação real do Backoffice local
- implementação real da Auth API local
- runbooks locais validados
- ajustes incrementais recentes do fluxo session-first

### Fontes de apoio histórico

- master antigo
- blueprint legado
- documentos anteriores do Backoffice ainda úteis como apoio histórico
- referências antigas sobre separação entre Portal, Auth API e camadas administrativas

Regra documental:

- histórico ajuda a reconciliar
- mas não governa o contexto sozinho

---

## Artefatos reais do Backoffice

Os artefatos reais já materializados neste contexto incluem:

### Aplicação Angular

- SPA administrativa própria
- Angular 21
- standalone-first
- TypeScript
- SCSS
- routing ativo

### Root files conhecidos

- `src/main.ts`
- `src/app/app.component.ts`
- `src/app/app.component.html`
- `src/app/app.component.scss`
- `src/app/app.config.ts`
- `src/app/app.routes.ts`

### Configuração local de proxy

- `proxy.conf.json`

### Script local de start

- `npm start`
- com `ng serve --proxy-config proxy.conf.json`

### Estrutura principal da aplicação

- `src/app/core/`
- `src/app/features/`
- `src/app/shared/` quando aplicável

---

## Estrutura principal conhecida

A estrutura principal reconciliada do Backoffice já inclui, no mínimo:

```text id="n0h1h0"
src/app/
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
    interceptors/
    layout/
      admin-shell/
      header/
      sidebar/
      page-container/

  features/
    auth/
      pages/
        login-page/
    dashboard/
      pages/
        dashboard-page/
    error-pages/
      forbidden-page/
      not-found-page/
```

Essa estrutura já é suficiente para:

* boot do app
* navegação protegida
* resolução de sessão
* shell administrativo
* UX local de desenvolvimento do admin

---

## Componentes estruturais relevantes

Os componentes mais importantes já reconciliados neste contexto são:

### `AuthApiService`

Papel:

* encapsular contratos administrativos de auth
* consumir `GET /auth/session`
* consumir `POST /auth/dev/bootstrap-session` em desenvolvimento local

### `AuthSessionStore`

Papel:

* manter estado de sessão
* expor Signals e `computed()`
* resolver sessão real
* recarregar sessão explicitamente
* sustentar guards e UI

### `auth.guard`

Papel:

* bloquear rota protegida sem sessão válida
* aguardar resolução assíncrona da sessão antes de decidir

### `admin-access.guard`

Papel:

* distinguir `unauthenticated` de `forbidden`
* controlar acesso administrativo mínimo

### `admin-shell`

Papel:

* moldura protegida da área administrativa
* organizar sidebar, header e conteúdo sob sessão válida

### `login-page`

Papel:

* superfície inicial do admin local
* bootstrap local de sessão
* resolução explícita de sessão atual
* exibição de erro e status técnico de auth

### `dashboard-page`

Papel:

* superfície técnica inicial do PR-1
* refletir store, status, operador e role
* ajudar na validação do fluxo session-first

---

## Contratos administrativos reais já consumidos

Os contratos já materializados e relevantes para o Backoffice incluem:

### Sessão administrativa atual

* `GET /auth/session`

Estado:

* real
* já consumido pela store de auth
* já usado por guards assíncronos

### Bootstrap local de sessão

* `POST /auth/dev/bootstrap-session`

Estado:

* real
* auxiliar de desenvolvimento
* não é contrato normal de produto

### Superfícies administrativas de News

Estado:

* reais
* já existentes na Auth API
* prontas para futura integração de domínio no Backoffice

### Superfícies administrativas de Seasons

Estado:

* reais para escrita e lifecycle
* prontas para evolução futura do primeiro domínio forte do admin

---

## Fluxo local real já validado

O fluxo local reconciliado deste contexto já inclui:

1. subir a Auth API local
2. subir o Backoffice local com proxy
3. abrir `/login`
4. criar sessão local de desenvolvimento
5. navegar para `/dashboard`
6. recarregar sessão pela UI quando necessário
7. atualizar a página em `/dashboard`
8. manter sessão válida após refresh

Esse fluxo já é parte do inventário do contexto porque foi validado ponta a ponta.

---

## Paths críticos do contexto

Os paths mais importantes do Backoffice, nesta fase, incluem:

### Root do projeto

* repositório local do Backoffice
* diretório base da SPA administrativa

### Arquivos de bootstrap

* `src/main.ts`
* `src/app/app.config.ts`
* `src/app/app.routes.ts`

### Auth local

* `src/app/core/auth/auth-api.service.ts`
* `src/app/core/auth/auth-session.store.ts`
* `src/app/core/auth/models/auth.model.ts`
* `src/app/core/auth/models/role.model.ts`

### Guards

* `src/app/core/guards/auth.guard.ts`
* `src/app/core/guards/admin-access.guard.ts`

### Layout

* `src/app/core/layout/admin-shell/`
* `src/app/core/layout/header/`
* `src/app/core/layout/sidebar/`
* `src/app/core/layout/page-container/`

### Páginas iniciais

* `src/app/features/auth/pages/login-page/`
* `src/app/features/dashboard/pages/dashboard-page/`
* `src/app/features/error-pages/forbidden-page/`
* `src/app/features/error-pages/not-found-page/`

### Operação local

* `proxy.conf.json`
* `package.json`

Regra prática:

* qualquer mudança relevante nesses arquivos ou paths deve ser refletida nos docs canônicos do contexto

---

## Dependências externas relevantes

As dependências externas mais importantes do contexto são:

### Auth API

Dependência central do Backoffice.

Papel:

* autenticação administrativa
* introspecção de sessão
* autorização final
* backend de mutações administrativas

### Browser com cookie de sessão

Papel:

* manter sessão administrativa no ambiente local e, futuramente, publicado

### Proxy local do Angular

Papel:

* encaminhar `/auth`, `/admin`, `/content` e `/health`
* evitar fricção prematura com CORS/cookie no desenvolvimento

### Ambiente Node/npm local

Papel:

* subida do Angular
* validação da SPA
* iteração de desenvolvimento

---

## Comandos de validação rápida

Os comandos abaixo ajudam a validar rapidamente o contexto.

### Subir o Backoffice local

```bash id="9pdjmj"
cd ~/work/hsc/hsc-backoffice-admin
npm start
```

### Validar a Auth API local

```bash id="lqjofg"
curl -i http://127.0.0.1:3000/health
```

### Testar bootstrap de sessão pela Auth API

```bash id="5mab45"
curl -i -c /tmp/hsc-auth-cookie.txt -X POST http://127.0.0.1:3000/auth/dev/bootstrap-session
```

### Testar introspecção da sessão

```bash id="jssqni"
curl -i -b /tmp/hsc-auth-cookie.txt http://127.0.0.1:3000/auth/session
curl -i http://127.0.0.1:3000/auth/session
```

### Validar UX do Backoffice

Checklist manual mínimo:

* abrir `/login`
* clicar em `Criar sessão local de desenvolvimento`
* entrar em `/dashboard`
* dar refresh em `/dashboard`
* usar `Resolver sessão atual`
* usar `Recarregar sessão`

---

## Itens pendentes / zonas de atenção

Os itens abaixo ainda merecem atenção posterior.

### 1. Hostname final do Backoffice publicado

Estado:

* ainda não consolidado neste contexto

Impacto:

* afeta deploy final do frontend
* afeta reconciliação final de cookie/CORS em produção

### 2. Fluxo final de login administrativo de produção

Estado:

* bootstrap local existe
* login final de produção ainda não está fechado

Impacto:

* não bloqueia desenvolvimento local
* bloqueia o fechamento definitivo da jornada publicada

### 3. Política final de cookie/CORS no Backoffice publicado

Estado:

* local usa proxy
* produção ainda depende de reconciliação explícita

Impacto:

* afeta publicação final e refresh autenticado fora do ambiente local

### 4. Domínio `seasons` ainda não está materializado na SPA

Estado:

* backend já possui base real relevante
* frontend ainda não abriu PR-2 do domínio

Impacto:

* PR-1 está pronto como espinha dorsal
* valor de domínio ainda depende da próxima fase

### 5. Domínio `events` ainda não está reconciliado

Estado:

* ainda não maduro como contrato admin completo

Impacto:

* não deve ser tratado como pronto no frontend

---

## Itens explicitamente fora deste inventário

Os itens abaixo não pertencem ao inventário canônico do Backoffice Admin:

* portal público estático
* ETL Bash da v2
* servidor CS2
* AMP / Game Panel
* Nginx e systemd do Lightsail em profundidade
* detalhes completos de MariaDB
* credenciais ou secrets
* material legado não reconciliado

Esses assuntos pertencem a outros contextos ou devem permanecer fora do fluxo normal deste contexto.

---

## Limites documentais do contexto

Os limites documentais deste contexto são:

* ele documenta a SPA administrativa do HSC
* ele não documenta sozinho a Auth API
* ele não substitui governança nem índice mestre
* ele depende de reconciliação periódica com a implementação real para permanecer confiável
* ele deve separar claramente o que é fluxo local auxiliar do que é contrato normal de produto

---

## Critério de manutenção

Este documento deve ser atualizado quando houver:

* mudança de stack principal do frontend
* mudança relevante de estrutura da aplicação
* mudança de paths críticos
* mudança de contrato real de auth consumido pelo admin
* mudança do fluxo local de bootstrap ou resolução de sessão
* mudança relevante em guards, store ou shell
* criação de novo domínio admin materializado
* resolução de item pendente listado aqui
* mudança relevante do proxy local ou da ergonomia de operação local

---

## Critério de pronto

Este documento pode ser considerado maduro quando:

* a stack real do Backoffice estiver explícita sem ambiguidade
* a estrutura principal do app estiver claramente posicionada
* os contratos de auth relevantes estiverem classificados corretamente como reais ou auxiliares
* o fluxo local validado estiver registrado de forma reproduzível
* os paths críticos e dependências externas estiverem claros
* as pendências restantes estiverem explícitas
* ele puder servir como inventário confiável do contexto sem depender de memória informal

---

## Última revisão

* Status: ativo
* Classificação: canônico
* Contexto: backoffice admin / references and inventory
* Última revisão: 2026-03-19
