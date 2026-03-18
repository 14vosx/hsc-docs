# Auth, RBAC and Guards

## Objetivo

Documentar a estratégia canônica de autenticação, autorização, RBAC e proteção de rotas do Backoffice Admin do HSC.

Este documento existe para registrar, de forma estável e auditável:

- o modelo de autenticação administrativa do Backoffice
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
- guards de rota no Angular 20
- comportamento da UI diante de falta de sessão ou falta de permissão
- relação entre autorização de frontend e autorização de backend
- tratamento de contingência administrativa
- padrões mínimos de ocultação/bloqueio de ações por papel

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

- o Backoffice é uma SPA administrativa separada do Portal público
- a Auth API é a autoridade final para autenticação e autorização
- o modelo desejado do administrativo é session-first
- o fallback break-glass existe como contingência controlada no backend
- mutações administrativas sensíveis devem continuar fail-closed
- o frontend deve refletir permissões, nunca defini-las sozinho
- o Backoffice deve nascer pronto para RBAC real
- o fluxo principal do operador não deve depender de mecanismos emergenciais

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

A autenticação do Backoffice deve seguir o modelo:

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

O fluxo principal do Backoffice deve ser sempre session-first.

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
- publicar/despublicar quando aplicável
- ativar/fechar season
- excluir/cancelar quando o domínio permitir
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

## Estratégia Angular 20 para auth e guards

O Backoffice adota Angular 20 standalone-first.

Neste contexto, a estratégia recomendada é:

- estado de sessão em Signals
- guards funcionais
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

Exemplos de estado que devem existir:

- sessão ainda não resolvida
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

## Guards funcionais

Os guards do Backoffice devem ser simples, explícitos e funcionais.

Papéis centrais dos guards:

### `auth.guard`

Responsável por:

- impedir acesso a rotas protegidas sem sessão válida
- bloquear shell administrativo para usuário não autenticado

### `admin-access.guard`

Responsável por:

- exigir acesso administrativo mínimo
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
- um `editor` pode receber acesso a rotas de criação/edição
- um `admin` pode receber acesso a ações críticas e rotas mais sensíveis

Regra importante:

- ocultar menu por papel melhora ergonomia
- não é mecanismo de segurança

---

## Tratamento de 401

### Significado

- sessão ausente
- sessão expirada
- autenticação inválida
- identidade não pode ser resolvida a partir da sessão atual

### Comportamento esperado

- bloquear ou interromper fluxo protegido
- redirecionar para login quando apropriado
- limpar estado administrativo local inconsistente
- informar o operador de forma clara, quando necessário

### Regras de implementação

- o interceptor pode capturar 401 para reações transversais
- a store de sessão deve refletir o novo estado
- a UI não deve continuar mostrando shell “válido” após perda real de sessão

---

## Tratamento de 403

### Significado

- operador autenticado sem permissão suficiente
- rota aberta, mas ação rejeitada
- permissão derivada da UI desatualizada ou mais permissiva do que o backend

### Comportamento esperado

- exibir tela ou estado explícito de acesso negado
- não transformar 403 em erro genérico
- não simular sucesso parcial
- preservar clareza de que a negativa veio da autorização

### Regras de implementação

- rotas sensíveis podem redirecionar para `/403`
- ações pontuais podem exibir erro contextualizado na própria tela
- backend permanece source of truth

---

## Tratamento de erros de identidade e permissão

O Backoffice deve distinguir pelo menos quatro classes de problema:

### 1. Sem sessão

Exemplo:
- usuário não autenticado

Resposta:
- login

### 2. Sessão inválida ou expirada

Exemplo:
- sessão antiga, revogada ou inconsistente

Resposta:
- limpar estado
- redirecionar
- informar necessidade de novo login

### 3. Autenticado sem acesso administrativo

Exemplo:
- usuário autenticado, mas sem escopo para o Backoffice

Resposta:
- bloquear entrada no shell
- redirecionar para `/403` ou superfície equivalente

### 4. Autenticado com acesso parcial, mas sem permissão para ação específica

Exemplo:
- editor tentando executar ação de admin

Resposta:
- bloquear a ação na UI quando possível
- tratar 403 do backend de forma explícita

---

## Integração com interceptors

Os interceptors do Backoffice devem ser mínimos e previsíveis.

O que pode entrar em `auth.interceptor.ts`:

- headers e credenciais transversais
- preparação comum de requests administrativos
- política consistente de envio para a Auth API

O que pode entrar em `auth-error.interceptor.ts`:

- reação transversal a 401
- apoio à sincronização entre resposta HTTP e estado da store de sessão

O que não deve entrar em interceptor:

- regra de domínio de `news`
- regra de domínio de `seasons`
- regra de domínio de `events`
- autorização fina específica de feature

---

## Padrão de checagem de permissão na UI

A UI do Backoffice deve trabalhar com derivação simples de permissão.

Exemplos de uso:

- renderizar botão “Nova season” só para papel mínimo adequado
- ocultar ação “Publicar” quando o operador não tem escopo
- mostrar badge de “somente leitura” quando necessário
- bloquear ações destrutivas para papéis insuficientes

Regra importante:

- a checagem de permissão visual deve ser centralizada em signals derivados da store de sessão
- evitar espalhar comparações de papel soltas em muitos componentes

---

## Ações sensíveis

Ações sensíveis exigem disciplina adicional.

Exemplos:

- publish/unpublish de news
- activate/close de season
- delete/cancel de event
- exclusões e mudanças de lifecycle

Essas ações devem obedecer simultaneamente a:

- papel mínimo adequado
- confirmação explícita do operador
- resposta bem-sucedida do backend
- tratamento claro de erro
- ausência de optimistic update ingênuo no MVP

Regra importante:

- ação sensível não confirma visualmente antes do backend confirmar

---

## UX esperada por papel

### Viewer

A UI deve privilegiar:

- leitura
- listagem
- contexto
- ausência de controles de mutação sensível

### Editor

A UI deve privilegiar:

- criação
- edição
- fluxo de trabalho normal do domínio
- bloqueio claro de ações críticas reservadas a admin

### Admin

A UI deve privilegiar:

- acesso ao conjunto completo de ações administrativas
- confirmação de ações sensíveis
- feedback explícito sobre sucesso, falha e invariantes

---

## Riscos arquiteturais a evitar

### Risco 1 — tratar guard como autorização final

Efeito:
- falsa segurança
- backend e UI divergindo
- exposição indevida de ação

### Risco 2 — espalhar checagem de papel por toda a árvore de componentes

Efeito:
- manutenção difícil
- inconsistência
- bugs de permissão visual

### Risco 3 — misturar break-glass com login normal

Efeito:
- UX confusa
- erosão do modelo session-first
- acoplamento de produto à contingência

### Risco 4 — mascarar 403 como erro genérico

Efeito:
- operador não entende a causa real
- troubleshooting piora
- autorização fica opaca

### Risco 5 — modelar tudo como “admin ou não admin”

Efeito:
- perde nuance de RBAC
- reduz escalabilidade da camada administrativa
- dificulta evolução de domínios

### Risco 6 — deixar a UI otimista demais em ação sensível

Efeito:
- inconsistência visual
- falso positivo de sucesso
- colisão com fail-closed do backend

---

## Estrutura mínima recomendada

A estrutura mínima do Backoffice para auth, RBAC e guards deve ser algo como:

```text
src/app/
  core/
    auth/
      auth-api.service.ts
      auth-session.store.ts
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

- boot session-first
- resolução de identidade atual
- navegação protegida
- UI reativa por papel
- tratamento consistente de 401/403

---

## Ordem de implementação recomendada

A ordem recomendada para materializar essa camada é:

1. definir modelo de identidade, sessão e papel
2. criar `auth-api.service.ts`
3. criar `auth-session.store.ts` com Signals
4. implementar `auth.guard`
5. implementar `admin-access.guard`
6. implementar `role.guard` para rotas sensíveis
7. integrar interceptors de auth
8. adaptar shell e menu para papel atual
9. adaptar features para checagem derivada de permissão
10. validar 401, 403 e ações sensíveis com smoke funcional

---

## Resumo executivo

A camada de auth, RBAC e guards do Backoffice Admin do HSC deve ser construída sob o princípio:

- sessão primeiro
- backend como autoridade final
- frontend como refletor disciplinado de acesso e permissão

A implementação recomendada é:

- store transversal de sessão com Signals
- guards funcionais e simples
- RBAC explícito por domínio e ação
- menu e UI adaptados por papel
- 401 e 403 tratados como estados reais do produto
- break-glass mantido apenas como contingência operacional do backend

O Backoffice deve nascer pronto para RBAC real, sem depender de atalhos de UX, sem mascarar autorização e sem quebrar o modelo fail-closed do ecossistema HSC.