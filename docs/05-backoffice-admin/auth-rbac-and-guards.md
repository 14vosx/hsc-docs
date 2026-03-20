# Auth, RBAC and Guards

## Objetivo

Documentar a estratégia canônica de autenticação, autorização, RBAC e proteção de rotas do Backoffice Admin do HSC.

Este documento existe para registrar, de forma estável e auditável:

- o modelo de autenticação administrativa publicado do Backoffice
- a relação entre sessão, identidade e autorização
- o papel do RBAC na camada administrativa
- a função dos guards no frontend Angular
- a relação entre UI administrativa e autoridade do backend
- o comportamento esperado para 401, 403 e contingências administrativas
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
- callback com revalidação de sessão
- padrões mínimos de ocultação e bloqueio de ações por papel

Este documento não cobre em profundidade:

- contratos detalhados endpoint a endpoint
- implementação backend completa de sessão
- detalhes de banco para RBAC
- runbooks operacionais completos
- políticas de host, Nginx e deploy
- detalhes de auditoria persistida no backend

Esses tópicos vivem em outros documentos do contexto ou em contextos canônicos adjacentes.

---

## Estado atual

O estado arquitetural conhecido, nesta fase, é:

- o Backoffice já é uma SPA administrativa publicada separada do Portal público
- a Auth API é a autoridade final para autenticação e autorização
- o modelo administrativo publicado é session-first
- o login administrativo real usa magic link
- a sessão publicada é cookie-based e cross-subdomain
- o callback do frontend já revalida sessão antes de liberar o dashboard
- o fallback break-glass existe apenas como contingência controlada no backend
- mutações administrativas sensíveis continuam fail-closed
- o frontend deve refletir permissões, nunca defini-las sozinho
- o fluxo principal do operador não depende de mecanismo emergencial

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
- 401 e 403 tratados como estados reais de produto
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
Operador autenticado
  -> sessão válida
    -> identidade carregada
      -> papel/permissões resolvidas
        -> acesso à navegação e às ações administrativas conforme autorização
```

Leitura correta:

- primeiro existe a sessão
- depois existe a identidade do operador
- depois existem os papéis e permissões derivados
- só então a UI libera rotas e ações compatíveis

Regra importante:

- a UI não deve tentar inferir autorização antes de resolver o estado de sessão/identidade

---

## Session-first como fluxo principal

O fluxo principal do Backoffice é session-first.

Isso significa:

- o operador acessa o Backoffice sob sessão administrativa válida
- a aplicação carrega a identidade atual
- a aplicação resolve papel e permissões
- a aplicação libera ou bloqueia navegação e mutações conforme esse estado

O que session-first evita:

- dependência de chaves ou mecanismos excepcionais como fluxo normal
- UI aberta antes da validação real de acesso
- ambiguidade sobre quem é o operador atual
- acoplamento da UX administrativa a contingências operacionais

---

## Break-glass como contingência

O mecanismo break-glass continua existindo apenas como contingência operacional controlada.

Leitura correta do break-glass neste contexto:

- não é fluxo principal do Backoffice
- não deve ser tratado como jornada normal de login
- não deve aparecer como centro da UX administrativa
- deve permanecer restrito ao backend e aos runbooks operacionais apropriados
- seu uso deve seguir controle, observabilidade e auditabilidade do lado dinâmico

Regra importante:

- o Backoffice deve ser desenhado para funcionar corretamente sem depender do break-glass em uso normal

---

## Backend como autoridade final

A Auth API continua sendo a autoridade final para:

- autenticação
- resolução da identidade atual
- validação de papel
- validação de permissão
- aplicação de invariantes de domínio
- bloqueio de mutações indevidas
- auditoria

Consequências para o frontend:

- um botão visível não significa permissão garantida
- um guard aprovado não significa mutação garantida
- a UI deve tratar respostas do backend como decisão final
- 403 real do backend deve prevalecer sobre qualquer expectativa da interface

---

## Papel do frontend na autorização

O frontend tem três responsabilidades principais na camada de autorização:

### 1. Restringir entrada em rotas protegidas

Via guards de autenticação e guards de autorização mínima.

### 2. Ajustar a navegação e a UI ao papel do operador

Exemplos:

- esconder item de menu não aplicável
- ocultar ação sensível não disponível
- desabilitar controles quando a ação não faz sentido para aquele papel

### 3. Tratar rejeições reais do backend

Exemplos:

- exibir 403 de forma explícita
- impedir confirmação ilusória de ações falhadas
- manter consistência entre estado visual e decisão do servidor

O frontend não deve:

- assumir que o operador pode mutar porque a rota abriu
- decidir sozinho a autorização final da ação
- transformar erro de permissão em erro genérico e opaco

---

## Modelo inicial de RBAC

O Backoffice nasce pronto para RBAC explícito.

A hierarquia inicial mais coerente para o contexto administrativo é:

```text
viewer
editor
admin
```

### `viewer`

Papel de leitura administrativa.

Capacidades esperadas:

- acessar listagens administrativas permitidas
- visualizar dados e estados
- não realizar mutações sensíveis

### `editor`

Papel de edição controlada.

Capacidades esperadas:

- criar e editar entidades permitidas
- operar fluxos editoriais ou administrativos não críticos
- não executar lifecycle crítico ou ações destrutivas mais sensíveis, salvo decisão explícita de domínio

### `admin`

Papel administrativo sensível.

Capacidades esperadas:

- executar ações críticas
- publicar e despublicar quando aplicável
- ativar e fechar season
- excluir e cancelar quando o domínio permitir
- acessar superfícies administrativas mais restritas

---

## Nota de reconciliação com legado

O legado do HSC já aponta hierarquia próxima a `user < editor < admin`.

Para o Backoffice, a leitura recomendada é:

- `user` não é papel natural de operação administrativa
- no contexto da SPA administrativa, o papel prático de leitura pode ser modelado como `viewer`
- a reconciliação final entre nomenclatura backend e nomenclatura de UX deve ser explícita em contrato

Regra importante:

- se o backend expuser `user`, `editor`, `admin`, o frontend pode mapear isso para uma semântica administrativa mais clara
- esse mapeamento deve ser documentado e não pode gerar perda de segurança

---

## Matriz inicial de permissões por domínio

A matriz inicial recomendada é:

### News

- `viewer`: listar e visualizar
- `editor`: criar e editar draft
- `admin`: publicar, despublicar e excluir

### Seasons

- `viewer`: listar e visualizar
- `editor`: criar e editar metadados básicos
- `admin`: ativar e fechar season

### Events

- `viewer`: listar e visualizar
- `editor`: criar e editar evento
- `admin`: excluir, cancelar ou executar ações mais sensíveis

Regra importante:

- a permissão visual do frontend deve ser coerente com a matriz
- a validação definitiva continua no backend

---

## Estratégia Angular 21 para auth e guards

O Backoffice adota Angular 21 standalone-first.

Neste contexto, a estratégia recomendada é:

- estado de sessão em Signals
- guards funcionais e assíncronos quando dependem de sessão real
- providers globais em `app.config.ts`
- interceptors HTTP para comportamento transversal
- serviços de auth em `core/auth/`
- guards em `core/guards/`

Estrutura sugerida:

```text
src/app/
  core/
    auth/
      auth-session.store.ts
      auth-api.service.ts
      auth.model.ts
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

- carregar identidade atual
- manter status de sessão
- expor operador atual
- expor papel atual
- expor flags de inicialização
- expor flags de autenticado / não autenticado
- expor flags de acesso administrativo
- expor sinais derivados de permissão mínima
- oferecer `reloadSession()` como mecanismo explícito de revalidação

Exemplos de estado que devem existir:

- sessão ainda não resolvida (`unknown`)
- autenticado
- não autenticado
- carregando identidade atual
- identidade disponível
- papel atual disponível
- erro ao resolver sessão

Exemplos de derivados úteis com `computed()`:

- `isAuthenticated`
- `isAdminAreaAllowed`
- `currentRole`
- `canEditNews`
- `canPublishNews`
- `canActivateSeason`

Regra importante:

- permissões derivadas da UI são conveniência de renderização
- não substituem autorização real do backend

---

## Estados mínimos da sessão

A sessão administrativa deve ser tratada como máquina de estados simples.

Estados mínimos recomendados:

### `unknown`

A aplicação ainda não sabe se há sessão válida.

Uso:

- boot da aplicação
- refresh de página
- primeiro carregamento do shell protegido
- transição inicial após callback

### `authenticated`

Sessão válida e identidade disponível.

Uso:

- liberar navegação protegida
- resolver UI por papel
- permitir tentativas de mutação conforme papel

### `unauthenticated`

Sem sessão válida.

Uso:

- bloquear shell protegido
- redirecionar para login
- limpar estado administrativo local

### `error`

Falha relevante ao resolver o estado de sessão.

Uso:

- exibir erro explícito de autenticação/resolução
- permitir recuperação quando cabível
- evitar boot silencioso inconsistente

---

## Callback e revalidação de sessão

A superfície `/auth/callback` é parte real da jornada de login administrativo.

Responsabilidades do callback:

- ler `status` ou `error` da query string
- tratar erros como `invalid_or_expired_link`, `consume_failed`, `missing_token` e `forbidden`
- chamar `reloadSession()` quando `status=ok`
- tolerar pequeno race condition entre emissão do cookie e primeira consulta de sessão
- navegar para `/dashboard` apenas quando a sessão estiver validada

Regra importante:

- `status=ok` não significa autenticação confirmada por si só
- a autenticação só está concluída quando o backend confirmar a sessão

---

## Guards funcionais

Os guards do Backoffice devem ser simples, explícitos e funcionais.

Papéis centrais dos guards:

### `auth.guard`

Responsável por:

- impedir acesso a rotas protegidas sem sessão válida
- bloquear shell administrativo para usuário não autenticado
- aguardar resolução de sessão quando o estado ainda estiver `unknown`

### `admin-access.guard`

Responsável por:

- exigir acesso administrativo mínimo
- separar `unauthenticated` de `forbidden`
- impedir entrada em áreas administrativas para identidade sem escopo admin

### `role.guard`

Responsável por:

- exigir papel mínimo ou permissão mínima em rotas sensíveis
- proteger superfícies específicas quando necessário

Regra importante:

- não criar árvore excessivamente complexa de guards no MVP
- começar com poucos guards bem definidos

---

## Estratégia de rotas protegidas

A proteção recomendada de rotas segue esta ordem:

```text
/login                       -> pública
/auth/callback               -> pública com resolução de sessão
/dashboard                   -> auth + acesso administrativo
/seasons                     -> auth + acesso administrativo
/seasons/new                 -> auth + papel mínimo
/seasons/:slug/edit          -> auth + papel mínimo
/news                        -> auth + acesso administrativo
/news/new                    -> auth + papel mínimo
/news/:id/edit               -> auth + papel mínimo
/events                      -> auth + acesso administrativo
/events/new                  -> auth + papel mínimo
/events/:id/edit             -> auth + papel mínimo
/403                         -> pública
/404                         -> pública
```

Leitura recomendada:

- nem toda rota administrativa precisa exigir `admin`
- parte das superfícies pode exigir só autenticação administrativa
- rotas de mutação devem exigir papel mínimo coerente com o domínio

---

## Relação entre guards e UI

Guards e UI cumprem papéis diferentes.

### O guard faz

- bloquear entrada em rota inadequada
- impedir acesso estrutural à área protegida

### A UI faz

- ajustar menu
- mostrar ou ocultar ações
- desabilitar interações conforme permissão derivada
- tratar 403 do backend quando a ação ainda assim for tentada

Regra importante:

- um guard não substitui o controle fino da tela
- a tela não substitui o guard
- ambos não substituem o backend

---

## Menu e navegação por papel

A navegação administrativa deve refletir o papel do operador.

Comportamento esperado:

- menu não exibe entradas sem utilidade para o papel atual
- áreas sensíveis podem desaparecer ou ficar inacessíveis
- o shell continua estável, mas a superfície disponível se adapta

Exemplos:

- um `viewer` pode ver `seasons`, `news` e `events` apenas em modo leitura
- um `editor` pode acessar rotas de create/edit compatíveis
- um `admin` acessa lifecycle crítico e superfícies mais sensíveis

---

## 401, 403 e tratamento de erro

A UI administrativa deve tratar 401 e 403 como estados reais de produto.

### 401

Leitura:

- sessão inexistente, expirada ou inválida

Comportamento esperado:

- limpar estado administrativo local
- redirecionar para `/login` quando apropriado
- evitar loops silenciosos de tentativa infinita

### 403

Leitura:

- sessão existe, mas a permissão é insuficiente

Comportamento esperado:

- manter sessão consistente
- redirecionar para `/403` quando fizer sentido
- evitar mascarar rejeição de permissão como erro genérico

### 5xx

Leitura:

- falha sistêmica do backend ou infraestrutura

Comportamento esperado:

- mostrar erro útil
- permitir retry quando couber
- evitar interpretação indevida como ausência de sessão

---

## Critério de pronto

Este documento está suficientemente pronto quando:

- a jornada de auth publicada estiver corretamente refletida
- `reloadSession()` estiver reconhecido como peça central do frontend
- `/auth/callback` estiver documentado como superfície real do produto
- guards estiverem tratados como assíncronos quando dependem de sessão real
- backend permanecer documentado como autoridade final da sessão e da permissão

---

## Última revisão

- Revisado após a materialização do fluxo real de magic link admin publicado
- Revisado após a estabilização da sessão cookie-based cross-subdomain
- Revisado após a consolidação do callback `/auth/callback`
