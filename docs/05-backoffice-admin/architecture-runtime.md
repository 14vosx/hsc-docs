# Architecture Runtime

## Objetivo

Documentar a arquitetura real e alvo de runtime do contexto Backoffice Admin do ecossistema HSC, registrando sua topologia administrativa, os componentes envolvidos e as dependências operacionais que sustentam a camada de administração.

Este documento existe para registrar, de forma estável e auditável:

- a topologia do contexto Backoffice Admin
- os componentes reais e previstos que compõem essa camada
- a relação entre browser, shell administrativo, Auth API e domínios administrativos
- os princípios de sessão, autorização e mutação administrativa
- os principais failure domains do runtime do Backoffice
- os limites entre Backoffice, Portal e infraestrutura dinâmica

---

## Escopo

Este documento cobre:

- a visão geral da arquitetura do Backoffice Admin
- os componentes da cadeia administrativa
- o fluxo browser → Backoffice SPA → Auth API
- a relação do frontend administrativo com RBAC e sessão
- a topologia inicial dos domínios `news`, `seasons` e `events`
- os failure domains principais do produto administrativo
- as dependências centrais do runtime do Backoffice

Este documento não cobre em profundidade:

- detalhes de cada componente visual
- contratos endpoint a endpoint
- troubleshooting aprofundado
- deploy detalhado da Auth API
- configuração detalhada de Nginx
- ETL Bash e Static API v2
- operação do Game Panel
- detalhes profundos de banco de dados

Esses tópicos vivem em documentos próprios deste contexto ou em outros contextos canônicos.

---

## Estado atual

O estado conhecido do contexto `05-backoffice-admin`, nesta fase, é:

- o Backoffice Admin já é um alvo arquitetural explícito do ecossistema HSC
- a formalização canônica do contexto está começando agora
- a camada administrativa deve nascer separada do Portal público
- a Auth API já sustenta parte relevante das superfícies administrativas
- o modelo administrativo do backend já é session-first com fallback break-glass controlado
- mutações administrativas sensíveis já caminham sob postura fail-closed no backend dinâmico
- `news` e `seasons` possuem base documental e de contrato mais madura
- `events` já existe como domínio previsto, mas ainda pede reconciliação incremental
- a arquitetura do Backoffice deve priorizar MVP, baixo acoplamento e fronteiras claras

Regra importante:

- este documento registra tanto a realidade reconciliada disponível quanto o desenho-alvo imediato do produto administrativo
- onde a implementação ainda não estiver totalmente materializada, isso deve ser tratado como alvo arquitetural explícito, não como fato operacional concluído

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase, são:

- `docs/00-governance/documentation-system.md`
- `docs/00-governance/99-master-index.md`
- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/98-legacy/HSC_Master_Blueprint.md`
- `docs/98-legacy/HSC_MASTER_DOCUMENTATION.md`

O desenho do Backoffice nasce a partir da reconciliação entre:

- a separação de camadas já prevista no legado
- o estado atual da Auth API
- a necessidade de uma superfície administrativa própria
- a disciplina documental já adotada pelos contextos canônicos atuais

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/05-backoffice-admin/README.md`
- `docs/05-backoffice-admin/frontend-structure.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/operational-runbooks.md`
- `docs/05-backoffice-admin/references-inventory.md`

Este documento descreve a topologia e os fluxos do runtime do Backoffice.  
Ele não substitui:

- o documento de estrutura frontend
- o documento de RBAC e guards
- o documento de contratos administrativos
- os runbooks operacionais

---

## Visão arquitetural de alto nível

A topologia-alvo do Backoffice Admin é:

```text
Browser
  -> Backoffice SPA
    -> Auth API
      -> MariaDB / serviços internos do backend
```

Em paralelo:

```text
Browser
  -> Portal Estático
    -> Static API v2 / conteúdo público
```

Leitura correta dessa topologia:

- o Portal e o Backoffice são produtos distintos
- o Portal serve consumo público
- o Backoffice serve operação administrativa
- a Auth API é a camada dinâmica central para o administrativo
- o frontend administrativo não se comunica com ETL ou Static API v2 para mutações
- a autoridade de autenticação e autorização permanece no backend

---

## Papel do Backoffice na arquitetura do ecossistema

O Backoffice ocupa a camada administrativa do HSC.

Seu papel é:

- oferecer shell administrativo protegido
- centralizar navegação staff/admin
- apresentar listagens e formulários administrativos
- refletir permissões por domínio e ação
- acionar mutações administrativas na Auth API
- organizar os domínios administrativos em slices claros

O Backoffice não substitui:

- o Portal público
- a camada de publicação estática
- a Auth API como autoridade
- os runbooks da infraestrutura dinâmica

---

## Separação de camadas

A separação de camadas do ecossistema, para este contexto, deve ser lida assim:

- Portal público → consumo público
- Backoffice Admin → operação administrativa
- Auth API → backend dinâmico central
- Infra AWS Lightsail → runtime da Auth API
- Infra Hostinger / Portal Estático → camada pública estática e publishing
- Game Panel → operação do servidor CS2

Regra importante:

- o Backoffice não deve nascer acoplado ao Portal
- o Backoffice não deve depender de caminhos operacionais da camada estática para mutar domínio administrativo
- o Backoffice deve conversar com a Auth API por contrato explícito

---

## Componentes da cadeia administrativa

Os componentes principais desta arquitetura são:

### Browser

Responsável por:

- executar a SPA administrativa
- manter sessão do usuário
- renderizar shell, listagens e formulários
- refletir estados de loading, erro, vazio e sucesso
- obedecer decisões de autorização retornadas pelo backend

### Backoffice SPA

Responsável por:

- layout administrativo
- navegação entre domínios
- guards de rota
- leitura de sessão e papel do usuário
- integração com endpoints administrativos
- feedback transacional ao operador
- isolamento entre features administrativas

A SPA deve ser organizada por domínio e por componentes transversais mínimos, evitando acoplamento prematuro.

### Auth API

Responsável por:

- autenticação administrativa
- autorização real
- validação de RBAC
- aplicação das invariantes de domínio
- persistência das mutações administrativas
- auditoria
- postura fail-closed em mutações sensíveis

A Auth API continua sendo a autoridade final do sistema administrativo.

### Persistência e serviços internos do backend

Responsáveis por:

- armazenar dados administrativos
- manter estado de `news`, `seasons` e outros domínios
- sustentar auditoria administrativa
- suportar transações e invariantes críticas
- manter consistência entre mutação e observabilidade

Esta camada não é operada diretamente pelo Backoffice; ela é alcançada por contrato via Auth API.

---

## Fluxos principais do runtime

### Fluxo 1 — navegação administrativa

```text
Browser
  -> carrega Backoffice SPA
  -> resolve rota protegida
  -> consulta estado de sessão / identidade atual
  -> libera ou bloqueia acesso conforme autorização
```

Objetivo:

- impedir entrada indevida em áreas administrativas
- carregar shell apenas sob contexto consistente de autenticação

### Fluxo 2 — leitura administrativa

```text
Browser
  -> Backoffice SPA
    -> Auth API (leitura administrativa)
      -> resposta com dados e permissões aplicáveis
```

Objetivo:

- listar entidades administrativas
- carregar formulários de edição
- refletir estado atual de domínio sem inventar autoridade no frontend

### Fluxo 3 — mutação administrativa

```text
Browser
  -> ação do operador
    -> Backoffice SPA
      -> Auth API
        -> valida sessão / permissão / invariantes
        -> executa mutação
        -> persiste audit quando aplicável
        -> responde sucesso ou erro
```

Objetivo:

- manter mutação administrativa sob autoridade do backend
- alinhar UX do admin com regras reais do domínio

Regra importante:

- a UI não assume sucesso por antecipação
- a confirmação deve depender da resposta explícita do backend

### Fluxo 4 — contingência administrativa

```text
Operação controlada
  -> caminho break-glass no backend
    -> acesso administrativo emergencial
```

Leitura correta:

- o caminho break-glass existe para contingência operacional
- ele não deve ser modelado como fluxo principal de UX do Backoffice
- o frontend deve privilegiar o fluxo session-first
- o uso de break-glass deve continuar auditável e controlado na camada dinâmica

---

## Princípios de autenticação e autorização

O Backoffice deve nascer sob os seguintes princípios:

- session-first
- autorização real no backend
- frontend refletindo permissão, nunca substituindo permissão
- 401 e 403 tratados como estados explícitos de produto
- rotas protegidas por guard
- ações sensíveis protegidas por checagem de papel e por resposta do backend
- break-glass mantido como contingência e não como jornada principal

A hierarquia inicial esperada continua sendo:

- `user`
- `editor`
- `admin`

Leitura inicial dessa hierarquia no contexto administrativo:

- `user` não é papel natural de mutação administrativa
- `editor` representa edição controlada
- `admin` representa ações sensíveis e lifecycle crítico

---

## Domínios administrativos iniciais

### News

Topologia funcional esperada:

- listagem administrativa
- criação
- edição
- publicação
- despublicação
- exclusão

Leitura arquitetural:

- domínio editorial com maior maturidade inicial
- bom candidato para validar padrões de listagem, formulário e ações críticas

### Seasons

Topologia funcional esperada:

- listagem
- criação
- edição
- ativação
- fechamento

Leitura arquitetural:

- domínio com invariantes mais explícitas
- bom candidato para validar ações críticas de lifecycle
- domínio prioritário para o MVP do Backoffice

### Events

Topologia funcional esperada:

- listagem
- criação
- edição
- exclusão ou cancelamento

Leitura arquitetural:

- domínio administrativo previsto
- deve nascer no Backoffice como agenda administrativa
- confirmação de presença por usuário final não deve ser tratada como fluxo principal do Backoffice

---

## Estrutura lógica recomendada do frontend

A estrutura lógica inicial recomendada para a SPA é:

```text
src/app/
  core/
  shared/
  features/
    dashboard/
    news/
    seasons/
    events/
```

Leitura dessa estrutura:

### `core/`

Camada transversal:

- auth
- sessão
- guards
- interceptors
- config
- layout

### `shared/`

Camada de reaproveitamento:

- UI base
- tabela
- feedback
- formulários
- utilitários
- modelos compartilhados

### `features/`

Camada por domínio:

- `dashboard`
- `news`
- `seasons`
- `events`

Regra importante:

- o Backoffice deve ser organizado por slice de domínio
- não por uma massa genérica de telas administrativas

---

## Failure domains principais

Os principais failure domains desta arquitetura são:

### Falha de sessão

Efeitos:

- usuário perde acesso administrativo
- rotas protegidas deixam de funcionar
- ações sensíveis falham

Tratamento esperado:

- redirecionamento consistente
- feedback claro
- diferenciação entre sessão ausente e sessão expirada, quando possível

### Falha de autorização

Efeitos:

- usuário autenticado acessa superfície sem permissão suficiente
- ações críticas retornam 403
- UI e backend entram em desacordo de expectativa

Tratamento esperado:

- tela ou estado explícito de acesso negado
- componentes ocultando ações não permitidas
- backend permanecendo autoridade final

### Falha de contrato frontend/backend

Efeitos:

- listagens quebram
- formulários não carregam
- mutações falham por shape inesperado

Tratamento esperado:

- contratos explícitos
- mapeamento defensivo
- mensagens de erro úteis
- reconciliação documental rápida

### Falha transacional em mutações sensíveis

Efeitos:

- ação administrativa não deve confirmar
- regra fail-closed deve bloquear efeito parcial
- operador pode interpretar erro como bug de UI quando a causa real está na camada dinâmica

Tratamento esperado:

- UI não otimista para ações críticas
- feedback claro
- observabilidade do backend como apoio obrigatório

### Falha de separação de camadas

Efeitos:

- tentativa de resolver mutação pelo Portal ou pela camada estática
- acoplamento incorreto entre produtos
- erosão da arquitetura

Tratamento esperado:

- reforçar contrato Backoffice ↔ Auth API
- não abrir atalhos pela camada pública
- documentar claramente os limites do contexto

---

## Dependências operacionais centrais

As dependências operacionais centrais do Backoffice são:

- disponibilidade da Auth API
- integridade do modelo de sessão administrativa
- coerência do RBAC no backend
- contratos administrativos estáveis
- observabilidade mínima para troubleshooting
- política de origem / integração adequada entre frontend administrativo e backend

Embora o Backoffice seja um produto frontend, sua estabilidade depende diretamente da camada dinâmica.

---

## Invariantes arquiteturais

As invariantes arquiteturais deste contexto são:

- o Backoffice é separado do Portal público
- o Backoffice consome a Auth API
- a Auth API é a autoridade final de autenticação e autorização
- mutações administrativas sensíveis não devem contornar a postura fail-closed do backend
- break-glass não é UX principal
- o frontend deve ser organizado por domínio
- o MVP deve privilegiar simplicidade, clareza e auditabilidade

---

## Ordem de implementação recomendada

A ordem de implementação mais coerente, neste contexto, é:

1. shell administrativo
2. auth state, guards e tratamento de 401/403
3. `seasons`
4. `news`
5. `events`
6. acabamento transversal e runbooks

Essa ordem existe para:

- validar a base arquitetural primeiro
- começar por domínio com invariantes fortes
- aproveitar superfícies já maduras
- deixar o domínio ainda mais aberto por último

---

## Resumo executivo

O runtime do Backoffice Admin deve ser lido como:

- uma SPA administrativa separada do Portal
- dependente da Auth API como camada dinâmica central
- organizada por domínios administrativos
- subordinada a sessão, RBAC e fail-closed do backend
- evoluída incrementalmente a partir de `seasons`, `news` e `events`

Este contexto existe para garantir que a camada administrativa do HSC nasça com fronteira clara, integração disciplinada e evolução sustentável.