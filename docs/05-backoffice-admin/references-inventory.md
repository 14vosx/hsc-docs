# References Inventory

## Objetivo

Documentar o inventário de referências, dependências, artefatos, integrações e gaps do contexto `05-backoffice-admin` do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- as referências canônicas que governam o Backoffice Admin
- os contextos e documentos dos quais este contexto depende
- os artefatos técnicos previstos para a SPA administrativa
- as integrações centrais do Backoffice com o restante do ecossistema
- os recursos e superfícies que precisam ser reconciliados com o runtime real
- os gaps conhecidos, pendências e zonas de atenção do contexto

---

## Escopo

Este documento cobre:

- referências documentais do Backoffice Admin
- dependências arquiteturais do contexto
- inventário lógico de artefatos da SPA administrativa
- integrações com Auth API e contextos adjacentes
- inventário inicial de domínios administrativos
- pontos de reconciliação com legado
- gaps conhecidos de contrato, runtime e operação
- validação de fronteira do contexto

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
- quais limites este contexto não pode ultrapassar?

Regra importante:

- este documento não substitui o `README.md`
- ele complementa o contexto com visão de inventário e reconciliação

---

## Estado atual do contexto

O estado atual conhecido do contexto é:

- o contexto `05-backoffice-admin` está sendo formalizado agora
- o Backoffice Admin é uma SPA administrativa separada do Portal público
- a Auth API é a dependência dinâmica central do contexto
- o desenho assume Angular 20 + TypeScript + Signals
- o modelo de auth do admin é session-first
- o backend permanece autoridade final para autenticação, autorização e invariantes
- `seasons`, `news` e `events` são os domínios iniciais
- parte das superfícies administrativas já possui base mais madura
- parte do contexto ainda depende de reconciliação com o runtime real

Regra importante:

- este documento deve distinguir claramente:
  - o que já é referência canônica consolidada
  - o que é artefato-alvo do Backoffice
  - o que ainda é gap ou pendência

---

## Referências canônicas diretas deste contexto

Os documentos canônicos diretos de `05-backoffice-admin` são:

- `docs/05-backoffice-admin/README.md`
- `docs/05-backoffice-admin/architecture-runtime.md`
- `docs/05-backoffice-admin/frontend-structure.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/operational-runbooks.md`
- `docs/05-backoffice-admin/references-inventory.md`

Leitura recomendada:

- `README.md` define a fronteira do contexto
- `architecture-runtime.md` define a topologia
- `frontend-structure.md` define a estrutura da SPA
- `auth-rbac-and-guards.md` define acesso e proteção
- `admin-api-contracts.md` define contratos com a Auth API
- `operational-runbooks.md` define validação e troubleshooting
- este arquivo consolida o inventário de referência, dependência e fronteira

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

Papel dessas referências:

- registrar o runtime dinâmico real consumido pelo Backoffice
- sustentar o modelo session-first
- sustentar fallback break-glass como contingência backend
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

## Referências históricas e de apoio

Além das referências canônicas obrigatórias, este contexto pode depender de materiais históricos ou de implantação que ajudem na reconciliação de runtime.

Exemplos típicos:

- docs de PR ou HSC_IMPL relevantes
- notas de implantação que afetem contratos admin
- documentos de MVP de domínio
- backlog de features administrativas
- registros de gaps encontrados durante implementação

Regra importante:

- documentos de apoio não substituem o contexto canônico
- quando um documento de apoio alterar entendimento estrutural, o contexto 05 deve ser atualizado

---

## Validação de fronteira do contexto

A validação de fronteira deste contexto confirma que `05-backoffice-admin`:

- não cobre operação de host
- não cobre runtime profundo da Auth API
- não cobre Portal público estático
- não cobre ETL / publicação de conteúdo público
- não cobre fluxos públicos de usuário final

Leitura correta da fronteira:

- o Backoffice cobre a SPA administrativa e suas dependências imediatas de frontend, contrato e operação funcional
- os demais aspectos permanecem documentados nos contextos `01`, `03` e `04`
- a separação de camadas é parte central do inventário do contexto, e não apenas uma convenção textual solta

---

## Inventário lógico da aplicação Backoffice

A SPA administrativa prevista por este contexto deve conter, no mínimo, os seguintes artefatos lógicos.

### Entrada e configuração

- `src/main.ts`
- `src/app/app.component.ts`
- `src/app/app.config.ts`
- `src/app/app.routes.ts`

Papel:

- bootstrap Angular 20 standalone
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
- helpers utilitários
- tipos neutros

---

### Features

Slices iniciais previstos:

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

## Dashboard

Papel esperado:

- entrada autenticada do Backoffice
- validação de shell, sessão e navegação
- superfície técnica inicial
- não precisa nascer como dashboard analítico sofisticado

Status do domínio no contexto:

- domínio estrutural do produto
- prioridade alta para o shell inicial

---

## Seasons

Papel esperado:

- domínio administrativo prioritário do MVP
- validar listagem, create/edit e lifecycle sensível
- exercitar activate/close com invariantes de domínio

Superfícies administrativas esperadas:

- listagem admin
- detail/edit admin
- create
- activate
- close

Status do domínio no contexto:

- mais maduro
- bom primeiro domínio para implementação

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

---

### Serviço de integração auth

Artefato esperado:

- `src/app/core/auth/auth-api.service.ts`

Papel:

- chamadas de sessão/identidade
- apoio ao login/logout, quando aplicável
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

- necessário para qualquer release do Backoffice
- rota final ainda pode precisar de reconciliação com a release ativa

---

### Leitura administrativa

Objetivo:

- popular listagens
- carregar formulários
- obter estados de domínio

Status:

- `news` e parte de `seasons` têm base mais madura
- leituras admin completas ainda podem exigir confirmação por release

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

Papel deste inventário:

- orientar stores, guards e UI
- garantir que o produto trate estados reais
- evitar que erro de domínio seja tratado como erro genérico

---

## Inventário de artefatos de UI esperados

Componentes/estruturas transversais que o Backoffice deve prever:

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

- Angular 20
- TypeScript
- standalone components
- Angular Router
- HttpClient
- Signals
- RxJS

Leitura dessa stack:

- Angular 20 é a base da SPA
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

Superfícies que já aparecem com maior clareza na base atual.

Exemplos:

- trilha administrativa de `news`
- mutações principais de `seasons`
- fundamentos de auth/admin na Auth API

### Superfícies-alvo

Superfícies necessárias para o Backoffice operar bem, mas ainda sujeitas a reconciliação fina por release.

Exemplos:

- leitura admin canônica de `seasons`
- leitura admin canônica de `events`
- shape final da rota de identidade/sessão
- lifecycle final de `events` no admin

Regra importante:

- o documento deve evitar tratar superfície-alvo como superfície já materializada

---

## Inventário de gaps conhecidos

Os principais gaps atuais do contexto são:

### Gap 1 — rota final de sessão/identidade

Estado:

- necessidade funcional clara
- shape e rota final ainda dependem de reconciliação com a release ativa

Impacto:

- afeta boot do shell
- afeta guards
- afeta menu por papel

---

### Gap 2 — leitura admin de Seasons

Estado:

- domínio maduro
- mutações mais claras do que a superfície de leitura admin formalizada

Impacto:

- afeta listagem
- afeta tela de edição
- afeta qualidade do MVP do admin

---

### Gap 3 — leitura admin de Events

Estado:

- domínio previsto
- superfície admin ainda mais imatura

Impacto:

- afeta viabilidade do CRUD admin completo
- pode exigir formalização adicional antes da implementação

---

### Gap 4 — fronteira final de Events entre admin e usuário final

Estado:

- confirmação de presença aparece no histórico do domínio
- separação semântica precisa ficar mais clara

Impacto:

- risco de acoplamento incorreto
- risco de UX administrativa assumir papel que pertence ao Portal/Account

---

### Gap 5 — hostname/runtime final do Backoffice

Estado:

- contexto documental já nasceu
- superfície operacional de publicação ainda pode não estar fechada

Impacto:

- afeta runbooks
- afeta smoke pós-release
- afeta ambiente e deploy

---

### Gap 6 — telemetria e observabilidade do frontend

Estado:

- ainda não formalizada neste contexto

Impacto:

- troubleshooting depende fortemente de console/network + logs da Auth API
- pode exigir amadurecimento futuro

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
- a fronteira do contexto voltar a ficar implícita em vez de explícita

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
- a estrutura Angular 20 estiver fixada
- a estratégia de auth/RBAC estiver explícita
- os runbooks mínimos estiverem definidos
- os gaps estiverem visíveis e não escondidos
- a validação de fronteira estiver registrada formalmente

A implementação do produto pode começar mesmo com gaps abertos, desde que:

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
7. ao revisar se a fronteira do contexto continua correta

---

## Resumo executivo

O `references-inventory.md` do contexto `05-backoffice-admin` existe para manter uma visão consolidada de:

- referências canônicas
- dependências arquiteturais
- artefatos previstos da SPA
- integrações com a Auth API
- domínios administrativos do MVP
- gaps e riscos ainda abertos
- limites explícitos da fronteira do contexto

Ele deve ser lido como o mapa de inventário do Backoffice Admin do HSC:

- o que governa o contexto
- do que ele depende
- o que precisa existir
- o que já está maduro
- o que ainda precisa ser reconciliado
- o que este contexto deliberadamente não cobre
