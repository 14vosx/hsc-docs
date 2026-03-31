# Game Panel — README

## Objetivo

Documentar o contexto canônico do Game Panel do ecossistema HSC, incluindo a camada operacional do servidor CS2, o AMP Instance Manager, a instância oficial `MixHAXIXE01`, o MatchZy, os plugins instalados e a trilha administrativa do lado jogo baseada no CounterStrikeSharp.

Este contexto existe para registrar, de forma estável e auditável:

- o papel do Game Panel dentro do ecossistema HSC
- a relação entre AMP, instância, servidor CS2 e plugins
- os limites entre infraestrutura base, operação do jogo e publicação pública de dados
- a baseline administrativa canônica do lado jogo
- os procedimentos recorrentes de operação, manutenção e troubleshooting do servidor

---

## Navegação rápida

### Entrada
- [Home da documentação](../README.md)
- [Governance](../00-governance/README.md)
- [Master Index](../00-governance/99-master-index.md)

### Núcleo do contexto
- [AMP Instance Manager](./amp-instance-manager.md)
- [Architecture Runtime](./game-panel-architecture-runtime.md)
- [CS2 Server Configuration](./cs2-server-configuration.md)
- [Instance MixHAXIXE01](./instance-mixhaxixe01.md)
- [MatchZy](./matchzy.md)
- [MariaDB Runtime](./mariadb-runtime.md)
- [Plugins Installed](./plugins-installed.md)
- [Operational Runbooks](./game-panel-operational-runbooks.md)
- [Observability and Troubleshooting](./game-panel-observability-troubleshooting.md)
- [References and Inventory](./game-panel-references-inventory.md)
- [Public Access and Ops Entrypoint](./game-panel-public-access-and-ops-entrypoint.md)

### Trilha administrativa do lado jogo
- [Admin Authority Runtime](./game-panel-admin-authority-runtime.md)
- [Admin Operational Runbooks](./game-panel-admin-operational-runbooks.md)
- [Admin Command Profiles](./game-panel-admin-command-profiles.md)
- [Admin References Inventory](./game-panel-admin-references-inventory.md)
- [HSC — Estado Atual Oficial da Arquitetura Pública](../01-infra-hostinger/hsc-public-architecture-current-state.md)
- [Brand Hub Root — Product and Surface Decisions](../03-portal-estatico/brand-hub-root-product-and-surface-decisions.md)

### Relações com outros contextos
- [Infra Hostinger](../01-infra-hostinger/README.md)
- [Portal Estático](../03-portal-estatico/README.md)
- [Infra AWS Lightsail](../04-infra-aws-lightsail/README.md)
- [Backoffice Admin](../05-backoffice-admin/README.md)

### Trilhas operacionais relevantes
- [Data Sources — MatchZy SQLite](../03-portal-estatico/data-sources-matchzy-sqlite.md)
- [SQL Queries and Views](../03-portal-estatico/sql-queries-and-views.md)
- [ETL Bash Pipeline](../03-portal-estatico/etl-bash-pipeline.md)
- [Static API v2](../03-portal-estatico/static-api-v2.md)
- [Docker Host](../01-infra-hostinger/docker-host.md)

---

## Entry point operacional atual

O acesso operacional ao AMP / Game Panel não depende mais do apex público do domínio principal.

O entrypoint oficial atual do painel é:

- `https://ops.haxixesmokeclub.com`

O detalhamento dessa decisão e da restauração de acesso operacional vive em:

- [Game Panel — Public Access and Ops Entrypoint](./game-panel-public-access-and-ops-entrypoint.md)

---- 

## Escopo

Este contexto cobre exclusivamente a camada operacional do servidor CS2 sobre a base Hostinger.

Estão dentro do escopo deste contexto:

- AMP Instance Manager
- instância oficial `MixHAXIXE01`
- runtime do servidor Counter-Strike 2
- MatchZy
- plugins instalados
- configurações operacionais do servidor
- baseline administrativa do lado jogo
- runbooks de scrim, BO1, BO3, veto, coach, warmup e fallback admin
- troubleshooting funcional do lado jogo
- MariaDB do lado game, se e quando fizer parte do runtime real

Este contexto não cobre, em profundidade:

- Debian, Nginx, Docker host, Certbot e systemd como camada-base
- pipeline ETL Bash da Static API v2
- contratos JSON do portal
- frontend público do portal
- Auth API no AWS Lightsail
- SPA administrativa do Backoffice
- credenciais e arquivos sensíveis

---

## Estado atual

O estado operacional conhecido deste contexto é:

- o lado jogo do HSC opera sobre a infraestrutura Hostinger
- o AMP Instance Manager gerencia a instância oficial do ecossistema
- a instância oficial conhecida é `MixHAXIXE01`
- o servidor CS2 roda como workload operacional dessa instância
- o MatchZy é peça central do runtime do jogo
- há plugins instalados e relevantes para a operação do servidor
- o lado jogo produz a base de dados estatística que depois alimenta o Portal Estático
- a operação do Game Panel exige runbooks claros para evitar dependência de memória informal
- a autoridade administrativa do lado jogo foi centralizada no CounterStrikeSharp
- o grupo administrativo canônico ativo é `#hsc/root`
- o MatchZy e o CS2-SimpleAdmin deixaram de manter cadastros paralelos como fonte primária de autoridade

Leitura correta do estado atual:

- o Game Panel continua sendo a camada operacional viva do lado CS2 do HSC
- a administração do servidor e a administração da partida seguem separadas por responsabilidade
- a autoridade de admin agora tem baseline única e auditável
- a manutenção futura deve preservar essa topologia

---

## O que é o Game Panel no HSC

O Game Panel é a camada responsável por operar o servidor de jogo do ecossistema.

Seu papel é:

- sustentar a operação do CS2 em runtime
- permitir gestão prática da instância via AMP
- manter MatchZy e plugins funcionando
- sustentar fluxos de scrim e partida organizada
- sustentar a baseline administrativa do lado jogo
- produzir a base estatística operacional que alimenta a cadeia pública de dados do portal

Em termos arquiteturais, o Game Panel é o contexto que transforma a Infra Hostinger em um servidor de jogo efetivamente utilizável.

---

## Subtrilhas internas do contexto

A organização viva deste contexto, neste checkpoint, se divide em quatro subtrilhas.

### 1. Runtime estrutural do lado jogo

Cobre:

- AMP
- instância `MixHAXIXE01`
- servidor CS2
- filesystem e runtime de container associados

Documentos principais:

- `game-panel-architecture-runtime.md`
- `amp-instance-manager.md`
- `instance-mixhaxixe01.md`

### 2. Fluxo competitivo e operação de partida

Cobre:

- MatchZy
- warmup
- BO1 / BO3
- ready flow
- coach
- comandos de partida

Documentos principais:

- `matchzy.md`
- `cs2-server-configuration.md`
- `game-panel-operational-runbooks.md`

### 3. Plugins e superfícies auxiliares

Cobre:

- plugins carregados
- plugins core
- plugins administrativos
- plugins de suporte visual e funcional

Documentos principais:

- `plugins-installed.md`
- `player-model-changer.md`
- `player-model-changer-references-inventory.md`

### 4. Autoridade administrativa do lado jogo

Cobre:

- CounterStrikeSharp como fonte de verdade para admins
- CS2-SimpleAdmin como administração do servidor
- MatchZy como administração da partida
- baseline viva de grupos, admins e guardrails contra drift

Documentos principais:

- `game-panel-admin-authority-runtime.md`
- `game-panel-admin-operational-runbooks.md`
- `game-panel-admin-command-profiles.md`
- `game-panel-admin-references-inventory.md`

---

## Papel do Game Panel no ecossistema HSC

O Game Panel é a camada que opera o jogo e produz o runtime competitivo do HSC.

Seu papel no ecossistema é:

- manter a instância do servidor CS2 utilizável
- sustentar a experiência de scrim e partidas organizadas
- centralizar MatchZy e plugins do lado game
- preservar a estrutura da instância oficial
- centralizar a autoridade administrativa do lado jogo em baseline explícita
- gerar a base operacional de stats que alimenta o Portal Estático
- separar a lógica do jogo da infraestrutura base, da camada pública de publicação e da camada dinâmica de auth/admin

Em termos arquiteturais:

- Infra Hostinger = substrate
- Game Panel = operação do jogo
- Portal Estático = publicação pública dos dados
- AWS Lightsail = backend dinâmico da Auth API
- Backoffice Admin = superfície administrativa de produto

Essa separação reduz confusão entre “host”, “jogo”, “portal”, “backend dinâmico” e “produto administrativo”.

---

## Componentes centrais do contexto

Os componentes centrais já reconciliados deste contexto incluem:

### AMP Instance Manager

Papel:
- gerenciar a instância do lado game
- oferecer camada de administração da operação do servidor

### Instância `MixHAXIXE01`

Papel:
- representar a instância oficial do ecossistema
- sustentar a árvore operacional do runtime do jogo

### CounterStrikeSharp

Papel:
- base de plugins do lado jogo
- camada-base de autoridade administrativa
- ponto canônico de carga de `admins.json` e `admin_groups.json`

### Counter-Strike 2

Papel:
- workload principal do contexto
- servidor efetivamente utilizado pelos players e admins

### MatchZy

Papel:
- administrar lógica competitiva e estatística do runtime de partida
- atuar como peça central da operação do servidor e da geração de dados
- consumir a autoridade administrativa vinda do CounterStrikeSharp

### CS2-SimpleAdmin

Papel:
- administrar o servidor em runtime
- oferecer painel de staff, moderação e comandos utilitários
- consumir a autoridade administrativa vinda do CounterStrikeSharp

### Plugins auxiliares

Papel:
- complementar a operação do servidor
- oferecer capacidades administrativas, visuais ou funcionais adicionais

---

## Leitura recomendada por objetivo

### Entender o runtime do lado jogo

Ler nesta ordem:

1. `game-panel-architecture-runtime.md`
2. `amp-instance-manager.md`
3. `instance-mixhaxixe01.md`

### Entender partida, MatchZy e fluxo competitivo

Ler nesta ordem:

1. `matchzy.md`
2. `cs2-server-configuration.md`
3. `game-panel-operational-runbooks.md`

### Entender autoridade administrativa e manutenção de admins

Ler nesta ordem:

1. `game-panel-admin-authority-runtime.md`
2. `game-panel-admin-operational-runbooks.md`
3. `game-panel-admin-command-profiles.md`
4. `game-panel-admin-references-inventory.md`

### Entender troubleshooting e checkpoints vivos

Ler nesta ordem:

1. `game-panel-observability-troubleshooting.md`
2. `game-panel-references-inventory.md`
3. `game-panel-admin-references-inventory.md`

---

## Regra canônica deste contexto

No HSC, o contexto `02-game-panel` deve documentar o lado jogo com fronteiras claras.

Isso significa:

- não absorver documentação de host base que pertence ao contexto `01-infra-hostinger`
- não absorver documentação de ETL e publicação pública que pertence ao contexto `03-portal-estatico`
- não absorver documentação da Auth API e do backend dinâmico que pertence ao contexto `04-infra-aws-lightsail`
- não absorver documentação da SPA administrativa que pertence ao contexto `05-backoffice-admin`
- manter a autoridade administrativa do lado jogo como baseline explícita, e não como memória informal de plugin

---

## Estado documental desejado deste contexto

Este contexto é considerado documentalmente saudável quando:

- a topologia do runtime do lado jogo está clara
- a instância oficial está identificada
- MatchZy e plugins relevantes estão inventariados
- os runbooks operacionais do servidor estão explícitos
- a trilha administrativa do lado jogo está documentada e sustentável
- troubleshooting e checkpoints vivos estão fáceis de localizar

Quando isso deixa de ser verdade, este contexto deve ser revisado.
