# Auth, RBAC and Guards

## Objetivo

Documentar a estratégia canônica de autenticação, autorização, RBAC e proteção de rotas do Backoffice Admin do HSC.

Este documento existe para registrar, de forma estável e auditável:

- o modelo de autenticação administrativa do Backoffice
- a relação entre sessão, identidade e autorização
- o papel do RBAC na camada administrativa
- a função dos guards no frontend Angular
- a relação entre UI administrativa e autoridade do backend
- o comportamento esperado para `401`, `403` e contingências administrativas
- os limites entre fluxo normal de sessão e fallback break-glass

---

## Escopo

Este documento cobre:

- autenticação administrativa do Backoffice
- leitura e manutenção de sessão no frontend
- identidade do operador autenticado
- modelo de RBAC do contexto administrativo
- guards de rota no Angular 21
- comportamento da UI diante de falta de sessão ou falta de permissão
- relação entre autorização de frontend e autorização de backend
- tratamento de contingência administrativa
- padrões mínimos de ocultação e bloqueio de ações por papel

Este documento não cobre em profundidade:

- contratos detalhados endpoint a endpoint
- implementação backend completa de sessão
- detalhes de banco para RBAC
- runbooks operacionais completos
- políticas de host, Nginx e deploy
- detalhes de auditoria persistida no backend
- fluxo final de login administrativo de produção

Esses tópicos vivem em outros documentos do contexto ou em contextos canônicos adjacentes.

---

## Estado atual

O estado arquitetural conhecido, nesta fase, é:

- o Backoffice é uma SPA administrativa separada do Portal público
- a Auth API é a autoridade final para autenticação e autorização
- o modelo administrativo real já opera em session-first
- a introspecção de sessão administrativa já existe via `GET /auth/session`
- o fallback break-glass continua existindo como contingência controlada no backend
- mutações administrativas sensíveis devem continuar fail-closed
- o frontend deve refletir permissões, nunca defini-las sozinho
- o Backoffice já usa guards assíncronos para resolver sessão antes de decidir navegação
- o fluxo local de desenvolvimento já pode bootstrapar sessão admin de forma controlada

Regra importante:

- o Backoffice não deve modelar “acesso admin” como simples convenção de UI
- acesso administrativo é responsabilidade do backend, refletida pelo frontend

---

## Princípios do contexto

Os princípios centrais deste documento são:

- session-first
- backend como autoridade final
- frontend como refletor de estado e permissão
- guards como mecanismo de controle de entrada em rotas
- RBAC explícito por domínio e ação
- `401` e `403` tratados como estados reais de produto
- break-glass apenas como contingência
- fail-closed preservado para mutações sensíveis
- UX administrativa clara, sem falsa sensação de permissão

Princípio importante:

- o Backoffice pode esconder ações não permitidas por ergonomia
- isso não substitui a checagem real no backend

---

## Modelo canônico de autenticação administrativa

A autenticação do Backoffice segue o modelo:

```text
sessão administrativa real
  -> introspecção de sessão
  -> guards protegendo shell e rotas
  -> UI derivada do papel atual
```

A leitura correta é:

* a sessão administrativa é resolvida no frontend por contrato explícito
* a identidade atual vem do backend
* o papel atual vem do backend
* a UI reage ao papel, mas não o inventa
* a autorização final continua sendo aplicada na Auth API

### Superfície canônica de sessão

A superfície canônica para introspecção da sessão atual é:

* `GET /auth/session`

Comportamento esperado:

* `200` com `authenticated: true`, `user` e `role` quando houver sessão válida
* `401` com `authenticated: false` quando não houver sessão válida

Leitura importante:

* esse endpoint já deve ser tratado como **superfície real**
* o Backoffice não precisa mais operar com endpoint hipotético de sessão

---

## Relação entre sessão real e bootstrap local

O contexto administrativo agora distingue claramente dois fluxos:

### Fluxo normal do produto

* sessão administrativa real
* introspecção via `GET /auth/session`
* shell e guards liberados a partir desse contrato

### Fluxo auxiliar de desenvolvimento local

* `POST /auth/dev/bootstrap-session`
* criação controlada de sessão admin local
* uso somente em ambiente de desenvolvimento
* não é contrato normal de produto

Regra importante:

* o Backoffice pode usar bootstrap local para destravar desenvolvimento
* o Backoffice não deve modelar essa rota como login final de produção

---

## Backend como autoridade final

A Auth API continua sendo a autoridade final para:

* autenticação
* resolução da identidade atual
* validação de papel
* validação de permissão
* aplicação de invariantes de domínio
* bloqueio de mutações indevidas
* auditoria

Consequências para o frontend:

* um botão visível não significa permissão garantida
* um guard aprovado não significa mutação garantida
* a UI deve tratar respostas do backend como decisão final
* `403` real do backend deve prevalecer sobre qualquer expectativa da interface

---

## Papel do frontend na autorização

O frontend tem três responsabilidades principais na camada de autorização:

### 1. Restringir entrada em rotas protegidas

Via guards de autenticação e guards de autorização mínima.

### 2. Ajustar a navegação e a UI ao papel do operador

Exemplos:

* esconder item de menu não aplicável
* ocultar ação sensível não disponível
* desabilitar controles quando a ação não faz sentido para aquele papel

### 3. Tratar rejeições reais do backend

Exemplos:

* exibir `403` de forma explícita
* impedir confirmação ilusória de ações falhadas
* manter consistência entre estado visual e decisão do servidor

O frontend não deve:

* assumir que o operador pode mutar porque a rota abriu
* decidir sozinho a autorização final da ação
* transformar erro de permissão em erro genérico e opaco

---

## Modelo inicial de RBAC

O Backoffice deve nascer pronto para RBAC explícito.

A hierarquia inicial mais coerente para o contexto administrativo é:

```text
viewer
editor
admin
```

### `viewer`

Papel de leitura administrativa.

Capacidades esperadas:

* acessar listagens administrativas permitidas
* visualizar dados e estados
* não realizar mutações sensíveis

### `editor`

Papel de edição controlada.

Capacidades esperadas:

* criar e editar entidades permitidas
* operar fluxos editoriais ou administrativos não críticos
* não executar lifecycle crítico ou ações destrutivas mais sensíveis, salvo decisão explícita de domínio

### `admin`

Papel administrativo sensível.

Capacidades esperadas:

* executar ações críticas
* publicar e despublicar quando aplicável
* ativar e fechar season
* excluir e cancelar quando o domínio permitir
* acessar superfícies administrativas mais restritas

---

## Nota de reconciliação com legado

O legado do HSC já aponta hierarquia próxima a `user < editor < admin`. 

Para o Backoffice, a leitura recomendada é:

* `user` não é papel natural de operação administrativa
* no contexto da SPA administrativa, o papel prático de leitura pode ser modelado como `viewer`
* a reconciliação final entre nomenclatura backend e nomenclatura de UX deve ser explícita em contrato

Regra importante:

* se o backend expuser `user`, `editor`, `admin`, o frontend pode mapear isso para uma semântica administrativa mais clara
* esse mapeamento deve ser documentado e não pode gerar perda de segurança

---

## Matriz inicial de permissões por domínio

A matriz inicial recomendada é:

### News

* `viewer`: listar e visualizar
* `editor`: criar e editar draft
* `admin`: publicar, despublicar e excluir

### Seasons

* `viewer`: listar e visualizar
* `editor`: criar e editar metadados básicos
* `admin`: ativar e fechar season

### Events

* `viewer`: listar e visualizar
* `editor`: criar e editar evento
* `admin`: excluir, cancelar ou executar ações mais sensíveis

Regra importante:

* a permissão visual do frontend deve ser coerente com a matriz
* a validação definitiva continua no backend

---

## Estratégia Angular 21 para auth e guards

O Backoffice adota Angular 21 standalone-first.

Neste contexto, a estratégia recomendada é:

* estado de sessão em Signals
* guards funcionais assíncronos
* providers globais em `app.config.ts`
* interceptors HTTP para comportamento transversal
* serviços de auth em `core/auth/`
* guards em `core/guards/`

Estrutura sugerida:

```text
src/app/
  core/
    auth/
      auth-session.store.ts
      auth-api.service.ts
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
```

---

## Store de sessão administrativa

O estado de autenticação do Backoffice deve ficar concentrado em uma store transversal, por exemplo:

```text
core/auth/auth-session.store.ts
```

Responsabilidades dessa store:

* resolver a sessão atual
* manter status de sessão
* expor operador atual
* expor papel atual
* expor flags de inicialização
* expor flags de autenticado e não autenticado
* expor flags de acesso administrativo
* expor sinais derivados de permissão mínima
* permitir recarga explícita da sessão quando necessário

Exemplos de derivados úteis com `computed()`:

* `isAuthenticated`
* `isAdminAreaAllowed`
* `currentRole`
* `canEditNews`
* `canPublishNews`
* `canActivateSeason`

Regra importante:

* permissões derivadas da UI são conveniência de renderização
* não substituem autorização real do backend

---

## Estados mínimos da sessão

A sessão administrativa deve ser tratada como máquina de estados simples.

Estados mínimos recomendados:

### `unknown`

A aplicação ainda não sabe se há sessão válida.

Uso:

* boot da aplicação
* refresh de página
* primeiro carregamento do shell protegido

### `authenticated`

Sessão válida e identidade disponível.

Uso:

* liberar navegação protegida
* resolver UI por papel
* permitir tentativas de mutação conforme papel

### `unauthenticated`

Sem sessão válida.

Uso:

* bloquear shell protegido
* redirecionar para login
* limpar estado administrativo local

### `error`

Falha relevante ao resolver o estado de sessão.

Uso:

* indicar problema real de contrato ou backend
* manter o frontend em estado previsível
* evitar falsa conclusão de autenticação

---

## Guards do Backoffice

Os guards do Backoffice devem proteger a navegação em dois níveis:

### 1. Guard de autenticação

Responsável por:

* garantir que a sessão já foi resolvida
* permitir rota protegida só quando houver sessão válida
* redirecionar para `/login` quando não houver autenticação

Este guard deve operar de forma assíncrona, porque a decisão correta depende de introspecção real da sessão.

### 2. Guard de autorização mínima

Responsável por:

* verificar se o operador autenticado pode acessar a área administrativa
* separar `unauthenticated` de `forbidden`
* redirecionar para `/403` quando houver autenticação sem permissão suficiente

### 3. Guard por papel ou ação sensível

Responsável por:

* proteger rotas mais finas quando o produto exigir
* operar por papel, domínio ou ação
* impedir acesso indevido a superfícies mais sensíveis

---

## Guards assíncronos e resolução de sessão

Como `GET /auth/session` já é contrato real, os guards do Backoffice devem esperar a resolução da sessão antes de decidir.

Leitura correta:

* guard não deve assumir que `unknown` significa não autenticado
* guard não deve depender de o shell já ter carregado a sessão
* a decisão de navegação protegida deve acontecer após a store resolver a sessão

Isso evita:

* redirecionamento prematuro para `/login`
* flicker de navegação
* shell renderizando sob estado ambíguo

---

## Comportamento esperado para `401`

Quando o backend responder `401`, a leitura do frontend deve ser:

* sessão ausente, inválida, expirada ou não reconhecida
* store converge para `unauthenticated`
* rota protegida deve ser bloqueada
* o operador deve voltar para `/login`

O frontend não deve:

* transformar `401` em erro genérico opaco
* manter aparência de sessão ativa
* fingir autenticação parcial

---

## Comportamento esperado para `403`

Quando o backend responder `403`, a leitura do frontend deve ser:

* operador autenticado
* mas sem escopo suficiente para aquela superfície ou ação

Resposta recomendada da UI:

* exibir página ou estado explícito de acesso negado
* não mascarar como `404` genérico
* não transformar automaticamente em logout

---

## Relação entre guards e shell

O shell administrativo deve ser tratado como área protegida.

Leitura correta:

* o shell não deve ser a primeira camada a “descobrir” a sessão
* o shell deve ser liberado depois que a sessão foi resolvida
* header, sidebar e conteúdo já devem nascer sob estado auth consistente

Na prática:

* guards resolvem a entrada
* store sustenta o estado
* shell reflete o resultado

---

## Login local de desenvolvimento

No ambiente local, a página `/login` pode oferecer ações auxiliares para desenvolvimento, como:

* criar sessão local de desenvolvimento
* resolver sessão atual
* exibir status técnico da store
* exibir erro real de bootstrap de sessão

Leitura correta:

* isso é ergonomia de desenvolvimento
* não representa o fluxo final de autenticação de produção
* não altera os princípios de session-first e backend como autoridade

---

## Relação com break-glass

O fallback break-glass continua existindo no backend, mas o Backoffice não deve modelá-lo como UX principal.

Consequências:

* o operador normal não navega “logando com admin-key”
* o frontend não expõe jornada baseada em segredo operacional
* break-glass continua sendo assunto prioritariamente operacional da Auth API

Regra importante:

* o backend pode manter `x-admin-key`
* o frontend administrativo normal deve continuar session-first

---

## Riscos arquiteturais da camada

Os principais riscos desta camada são:

### Risco 1 — tratar UI como autorização real

Efeito:

* falsa sensação de segurança
* descolamento entre UX e backend

### Risco 2 — decidir auth cedo demais

Efeito:

* redirecionamento incorreto
* flicker
* perda de sessão aparente

### Risco 3 — misturar fluxo normal com contingência

Efeito:

* produto administrativo nasce poluído por mecanismo emergencial
* aumenta risco operacional

### Risco 4 — achatamento prematuro de papéis

Efeito:

* perde nuance de RBAC
* reduz escalabilidade da camada administrativa
* dificulta evolução de domínios

### Risco 5 — deixar a UI otimista demais em ação sensível

Efeito:

* inconsistência visual
* falso positivo de sucesso
* colisão com fail-closed do backend

---

## Estrutura mínima recomendada

A estrutura mínima do Backoffice para auth, RBAC e guards deve ser algo como:

```text
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
      role.guard.ts

    interceptors/
      auth.interceptor.ts
      auth-error.interceptor.ts
```

Essa estrutura já permite:

* boot session-first
* resolução de identidade atual
* navegação protegida
* UI reativa por papel
* tratamento consistente de `401` e `403`

---

## Ordem de implementação recomendada

A ordem recomendada para materializar essa camada é:

1. definir modelo de identidade, sessão e papel
2. criar `auth-api.service.ts`
3. criar `auth-session.store.ts` com Signals
4. integrar `GET /auth/session`
5. implementar `auth.guard` assíncrono
6. implementar `admin-access.guard`
7. implementar `role.guard` para rotas sensíveis
8. integrar interceptors de auth
9. adaptar shell e menu para papel atual
10. adaptar features para checagem derivada de permissão
11. validar `401`, `403`, refresh e ações sensíveis com smoke funcional

---

## Resumo executivo

A camada de auth, RBAC e guards do Backoffice Admin do HSC deve ser construída sob o princípio:

* sessão primeiro
* backend como autoridade final
* frontend como refletor disciplinado de acesso e permissão

A implementação recomendada é:

* store transversal de sessão com Signals
* introspecção real via `GET /auth/session`
* guards assíncronos e simples
* RBAC explícito por domínio e ação
* menu e UI adaptados por papel
* `401` e `403` tratados como estados reais do produto
* break-glass mantido apenas como contingência operacional do backend

O Backoffice deve nascer pronto para RBAC real, sem depender de atalhos de UX, sem mascarar autorização e sem quebrar o modelo fail-closed do ecossistema HSC.

---

## Última revisão

* Status: ativo
* Classificação: canônico
* Contexto: backoffice admin / auth, RBAC e guards
* Última revisão: 2026-03-19

