# Portal Estático — README

## Objetivo

Documentar o contexto canônico do Portal Estático do ecossistema HSC, incluindo sua arquitetura pública, a Static API v2, a cadeia de geração de dados e os limites operacionais dessa camada.

Este contexto existe para registrar, de forma estável e auditável:

- o papel do portal público dentro do ecossistema HSC
- a relação entre frontend estático e Static API v2
- a origem dos dados servidos ao usuário final
- a cadeia SQLite → ETL Bash → JSON estático → Nginx → browser
- os limites, responsabilidades e dependências desta camada

---

## Navegação rápida

### Entrada
- [Home da documentação](../README.md)
- [Governance](../00-governance/README.md)
- [Master Index](../00-governance/99-master-index.md)

### Documentos deste contexto
- [Architecture Runtime](./portal-estatico-architecture-runtime.md)
- [Static API v2](./static-api-v2.md)
- [Brand Hub Root — Product and Surface Decisions](./brand-hub-root-product-and-surface-decisions.md)
- [Brand Hub Root — Architecture Runtime](./brand-hub-root-architecture-runtime.md)
- [Brand Hub Root — Publishing and Cutover Runtime](./brand-hub-root-publishing-and-cutover-runtime.md)
- [Brand Hub Root — Brand and Logo System](./brand-hub-root-brand-and-logo-system.md)
- [Brand Hub Root — Frontend Implementation Runtime](./brand-hub-root-frontend-implementation-runtime.md)
- [Brand Hub Root — References and Inventory](./brand-hub-root-references-inventory.md)
- [ETL Bash Pipeline](./etl-bash-pipeline.md)
- [Data Sources — MatchZy SQLite](./data-sources-matchzy-sqlite.md)
- [SQL Queries and Views](./sql-queries-and-views.md)
- [JSON Contracts](./json-contracts.md)
- [Frontend Structure](./portal-estatico-frontend-structure.md)
- [Nginx Publishing and Cache](./nginx-publishing-cache.md)
- [Operational Runbooks](./portal-estatico-operational-runbooks.md)
- [Observability and Troubleshooting](./portal-estatico-observability-troubleshooting.md)
- [References and Inventory](./portal-estatico-references-inventory.md)

### Relações com outros contextos
- [Infra Hostinger](../01-infra-hostinger/README.md)
- [Game Panel](../02-game-panel/README.md)
- [HSC — Estado Atual Oficial da Arquitetura Pública](../01-infra-hostinger/hsc-public-architecture-current-state.md)
- [Game Panel — Public Access and Ops Entrypoint](../02-game-panel/game-panel-public-access-and-ops-entrypoint.md)
- [Infra AWS Lightsail](../04-infra-aws-lightsail/README.md)
- [Backoffice Admin](../05-backoffice-admin/README.md)

### Trilhas operacionais relevantes
- [MatchZy](../02-game-panel/matchzy.md)
- [Nginx Static Serving](../01-infra-hostinger/nginx-static-serving.md)
- [Systemd Automation](../01-infra-hostinger/systemd-automation.md)
- [Auth API Operations](../04-infra-aws-lightsail/auth-api-operations.md)

---

## Escopo

Este contexto cobre exclusivamente a camada de portal público estático e de publicação da Static API v2.

Estão dentro do escopo deste contexto:

- portal público estático do HSC
- Static API v2
- pipeline ETL Bash
- Brand Hub Root no apex como superfície pública principal
- separação explícita entre
  - root da marca
  - portal público
- fonte de dados baseada em `matchzy.db`
- contratos JSON publicados
- frontend em HTML, CSS modular e JavaScript com ESModules
- publicação via Nginx
- cache e espelho same-origin de conteúdo quando aplicável
- runbooks e troubleshooting ligados ao portal e à API estática

Este contexto não cobre, em profundidade:

- a VPS Hostinger como camada-base de infraestrutura
- AMP / Game Panel / operação de partida
- MatchZy como ferramenta operacional de jogo
- Auth API no AWS Lightsail
- credenciais e arquivos sensíveis
- backlog funcional e roadmap de produto

---

## Estado atual

O estado operacional conhecido deste contexto é:

- o portal público do HSC é 100% estático
- o portal não depende de backend dinâmico próprio para renderização
- a camada de dados pública usa Static API v2
- a Static API v2 é gerada por ETL Bash
- a fonte primária dos dados operacionais do lado game é o `matchzy.db`
- a publicação final é feita por Nginx
- o frontend usa HTML, CSS modular e JavaScript com ESModules
- o contexto não usa framework frontend como base da entrega pública atual

A arquitetura do Portal Estático existe para entregar:

- hub público da marca no root
- entrada institucional e narrativa do ecossistema
- ponte de marca para a superfície operacional do portal
- separação entre branding, leitura pública e operação administrativa
- continuidade da superfície pública CS2 em `/portal/cs2/`
- páginas públicas de consulta
- navegação leve
- consumo de JSONs estáticos previsíveis
- baixa complexidade operacional
- desacoplamento entre portal público e backend dinâmico

---

## O que é o Portal Estático HSC

O Portal Estático do HSC é a camada pública de visualização e consumo de dados do ecossistema.

Seu papel é:

- publicar interface pública do projeto
- consumir a Static API v2
- apresentar dados de ranking, partidas, mapas e jogadores
- expor experiência pública sem depender de SSR ou backend dinâmico dedicado nesta camada
- operar com baixo acoplamento em relação ao runtime do jogo

Em termos arquiteturais, o portal é uma SPA estática leve, servida por Nginx, com consumo de artefatos JSON gerados previamente.

---

## O que o Portal consome

O portal consome, prioritariamente:

- arquivos JSON publicados pela Static API v2
- artefatos derivados do `matchzy.db`
- dados processados pelo pipeline ETL Bash
- conteúdo same-origin espelhado, quando aplicável ao fluxo de News

Os recursos públicos consumidos ou relacionados a este contexto incluem, entre outros:

- `health.json`
- `ranking.json`
- `matches.json`
- `match/{id}.json`
- `maps.json`
- `map/{map}.json`
- `player/{steamid64}.json`
- `steam-cache/{steamid64}.json`

O frontend não deve assumir backend mutável ao vivo dentro do próprio contexto do portal.

---

## O que o Portal não faz

O Portal Estático não é responsável por:

- autenticação dinâmica
- operação administrativa principal
- persistência de sessão
- escrita em banco de dados
- execução de lógica administrativa de conteúdo
- geração de dados em tempo real dentro do browser
- operação direta do servidor CS2
- substituição da Auth API

Esses papéis pertencem a outros contextos canônicos do ecossistema.

---

## Relação com a Static API v2

A Static API v2 é a camada de dados pública deste contexto.

A relação entre portal e Static API v2 é:

- o ETL gera JSONs estáticos
- o Nginx publica esses JSONs
- o frontend consome esses JSONs por fetch
- a experiência pública depende da integridade e da compatibilidade desses contratos

Princípios operacionais dessa relação:

- o frontend depende de contratos estáveis
- o ETL deve ser determinístico
- a escrita dos JSONs deve evitar artefatos parciais
- o portal deve tratar a Static API v2 como fonte pública de dados deste contexto

---

## Relação com News same-origin

O contexto do Portal Estático pode operar com espelhamento same-origin de conteúdo de News.

Esse espelhamento existe para:

- simplificar consumo no frontend
- reduzir atrito entre camadas
- manter integração pública previsível quando a origem dinâmica do conteúdo pertence a outra camada do ecossistema

Regra importante:
- esse espelho não transforma o portal em backend dinâmico
- ele continua sendo parte da camada de publicação pública estática / semi-estática do ecossistema

---

## Brand Hub Root dentro deste contexto

O contexto `03-portal-estatico` também passa a documentar o **Brand Hub Root** publicado no apex:

- `https://haxixesmokeclub.com`

Os documentos específicos dessa superfície são:

- `brand-hub-root-product-and-surface-decisions.md`
- `brand-hub-root-architecture-runtime.md`
- `brand-hub-root-publishing-and-cutover-runtime.md`
- `brand-hub-root-brand-and-logo-system.md`
- `brand-hub-root-frontend-implementation-runtime.md`
- `brand-hub-root-references-inventory.md`

---

## Papel do ETL neste contexto

O ETL Bash é a ponte entre a fonte de dados operacional e a publicação pública da Static API v2.

Seu papel é:

- ler a base SQLite do contexto de jogo
- executar consultas e transformações
- gerar JSONs estáticos previsíveis
- atualizar o estado público da v2
- manter artefatos de saúde e consistência

Esse pipeline é parte central deste contexto porque o portal depende de seus outputs para operar corretamente.

---

## Papel do Nginx neste contexto

No contexto do Portal Estático, o Nginx é responsável por:

- servir o frontend estático
- servir os artefatos da Static API v2
- aplicar a camada pública de publishing
- sustentar o consumo same-origin quando aplicável
- proteger artefatos que não devem ser expostos publicamente

O Nginx participa da camada de entrega pública, mas não substitui o ETL nem a origem dos dados.

---

## Documentos canônicos deste contexto

Os documentos canônicos previstos para este contexto são:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/portal-estatico-architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/brand-hub-root-product-and-surface-decisions.md`
- `docs/03-portal-estatico/brand-hub-root-architecture-runtime.md`
- `docs/03-portal-estatico/brand-hub-root-publishing-and-cutover-runtime.md`
- `docs/03-portal-estatico/brand-hub-root-brand-and-logo-system.md`
- `docs/03-portal-estatico/brand-hub-root-frontend-implementation-runtime.md`
- `docs/03-portal-estatico/brand-hub-root-references-inventory.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/sql-queries-and-views.md`
- `docs/03-portal-estatico/json-contracts.md`
- `docs/03-portal-estatico/portal-estatico-frontend-structure.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/03-portal-estatico/portal-estatico-operational-runbooks.md`
- `docs/03-portal-estatico/portal-estatico-observability-troubleshooting.md`
- `docs/03-portal-estatico/portal-estatico-references-inventory.md`

Este `README.md` funciona como porta de entrada e índice local do contexto.

---

## Relação com outros contextos

Este contexto se relaciona diretamente com:

### `docs/01-infra-hostinger/`
Relação estrutural forte.  
A Infra Hostinger sustenta a base operacional de host, Nginx, filesystem, automação e publishing sobre os quais o Portal Estático existe.

### `docs/02-game-panel/`
Relação indireta, mas crítica.  
A camada Game Panel opera o runtime do jogo que produz os dados originais utilizados pela cadeia de publicação da v2.

### `docs/04-infra-aws-lightsail/`
Relação indireta e seletiva.  
A Auth API não é a base da Static API v2, mas pode influenciar conteúdos públicos específicos, como News, em fluxos de espelhamento ou integração.

### `docs/90-adr/`
Deve registrar decisões arquiteturais relevantes deste contexto, como consolidação da Static API v2 e separação entre portal público e backend dinâmico.

### `docs/95-impl-log/`
Deve registrar mudanças incrementais relevantes do portal, ETL, cache, contratos ou publishing.

### `docs/97-audit/`
Pode conter análises, gaps, inconsistências ou verificações ligadas ao portal e sua cadeia de dados.

### `docs/98-legacy/`
Preserva documentação histórica usada como base de migração deste contexto.

---

## Dependências externas

Este contexto depende de componentes externos e integrações relevantes, incluindo:

- dados produzidos a partir do runtime do jogo
- SQLite local com schema compatível
- pipeline ETL funcional
- filesystem e permissões corretas
- Nginx publicando corretamente
- browser consumindo JSONs compatíveis
- integridade dos contratos da Static API v2
- domínios e paths públicos coerentes com a topologia do ecossistema

Também há dependência de disciplina operacional para:

- evitar geração parcial de JSON
- manter ordem correta do pipeline
- preservar compatibilidade entre ETL e frontend

---

## Source of truth / evidências

As principais evidências que sustentam este contexto, nesta fase de migração documental, são:

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação específica do portal e da Static API v2
- documentação do ETL Bash e das queries versionadas
- impl-logs ligados a conteúdo, mirror same-origin e publicação

Enquanto a migração do contexto não estiver concluída, os documentos antigos seguem como fonte de extração e checagem cruzada, mas não como canônico final.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/99-master-index.md`
- `docs/00-governance/README.md`
- `docs/00-governance/documentation-system.md`

Dentro deste próprio contexto, ele é a porta de entrada para todos os documentos canônicos específicos do Portal Estático.

---

## Critério de pronto deste contexto

Este contexto poderá ser considerado formalmente migrado quando:

- a arquitetura do portal estiver documentada sem depender do master antigo
- a Static API v2 estiver formalizada como camada própria
- a origem dos dados estiver clara
- o pipeline ETL estiver descrito com ordem, responsabilidades e limites
- os contratos JSON estiverem centralizados
- a publicação via Nginx estiver documentada
- troubleshooting e runbooks estiverem centralizados
- os documentos antigos passarem a servir apenas como referência cruzada

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / porta de entrada do contexto
- Última revisão: 2026-03-18