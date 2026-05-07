# HSC Canonical Master Index

## Objetivo

Este documento é o índice mestre canônico do repositório documental do ecossistema HSC.

Ele existe para:

- apontar a estrutura oficial da documentação viva
- registrar o estado atual de maturidade dos contextos canônicos
- resumir os principais fatos operacionais já reconciliados no runtime real
- indicar quais pendências ainda existem de forma explícita
- coordenar a leitura transversal entre contextos
- evitar retorno ao modelo anterior de “master monolítico” como fonte primária de verdade

Regra canônica:

**a verdade documental do sistema vive por contexto, e não em um documento único gigante.**
Este índice coordena a leitura. Ele não substitui os documentos especializados.

---

## Navegação rápida

### Entrada global
- [Home da documentação](../README.md)
- [Governance](./README.md)
- [Documentation System](./documentation-system.md)

### Contextos canônicos
- [01 — Infra Hostinger](../01-infra-hostinger/README.md)
- [HSC — Estado Atual Oficial da Arquitetura Pública](../01-infra-hostinger/infra-hostinger-public-architecture-current-state.md)
- [02 — Game Panel](../02-game-panel/README.md)
  - [Game Panel — Public Access and Ops Entrypoint](../02-game-panel/game-panel-public-access-and-ops-entrypoint.md)
- [03 — Portal Estático](../03-portal-estatico/README.md)
  - [Brand Hub Root — Product and Surface Decisions](../03-portal-estatico/brand-hub-root-product-and-surface-decisions.md)
  - [Brand Hub Root — Publishing and Cutover Runtime](../03-portal-estatico/brand-hub-root-publishing-and-cutover-runtime.md)
  - [ETL Repository — Minimal Shell CI](../03-portal-estatico/etl-repository-minimal-shell-ci.md)
- [04 — Infra AWS Lightsail](../04-infra-aws-lightsail/README.md)
- [05 — Backoffice Admin](../05-backoffice-admin/README.md)
  - [News Admin API Contracts](../05-backoffice-admin/news-admin-api-contracts.md)
  - [News Admin Feature Implementation Spec](../05-backoffice-admin/news-admin-feature-implementation-spec.md)
  - [News Admin Frontend Implementation Runtime](../05-backoffice-admin/news-admin-frontend-implementation-runtime.md)
  - [News Functional Smoke Guide](../05-backoffice-admin/news-functional-smoke-guide.md)
  - [Seasons Admin Functional Smoke Guide](../05-backoffice-admin/seasons-admin-list-functional-smoke-guide.md)
  - [Backoffice UI Material Foundation](../05-backoffice-admin/backoffice-ui-material-foundation.md)

### Trilhas principais
- [Fluxo jogo → dados → portal](../03-portal-estatico/static-api-v2.md)
- [Fluxo admin → auth → backoffice](../04-infra-aws-lightsail/auth-api-operations.md)
- [Fluxo autoridade admin do lado jogo](../02-game-panel/game-panel-admin-authority-runtime.md)

### Histórico e auditoria
- [Impl Log](../95-impl-log/README.md)
  - [2026-04-02 — ETL Minimal Shell CI Workflow](../95-impl-log/2026-04-02-etl-minimal-shell-ci-workflow.md)
- [Audit](../97-audit/README.md)
- [Legacy](../98-legacy/README.md)

---

## Estado atual do repositório

A base documental canônica do HSC já está estruturada e utilizável.

Neste checkpoint, os cinco contextos canônicos ativos permanecem úteis como base viva de operação e produto:

- Infra Hostinger
- Game Panel / CS2
- Portal Estático / Static API v2
- Infra AWS Lightsail / Auth API
- Backoffice Admin

Leitura correta do estado atual:

- o repositório já pode ser tratado como **base canônica viva**
- a maior parte das pendências críticas de realidade operacional dos contextos de infraestrutura já foi fechada
- o Backoffice Admin deixou de ser apenas contexto preparatório e já possui auth base publicada, callback real e shell administrativo validado
- Seasons já possui listagem administrativa, criação em `draft`, edição de metadados e lifecycle básico de ativar/fechar funcionais no Backoffice, com contratos admin canônicos disponíveis na Auth API
- o contexto `02-game-panel` agora também possui trilha administrativa explícita, com autoridade centralizada no CounterStrikeSharp e runbooks próprios de manutenção
- o que resta agora é principalmente **expansão incremental de domínio, reconciliação fina entre canônicos e evolução sustentável das superfícies administrativas**

---

## Estrutura canônica

## 00-governance

Função:
- governança documental
- regras do sistema documental
- índice mestre canônico

Arquivos principais:
- [Governance — README](./README.md)
- [Documentation System](./documentation-system.md)
- [HSC Docs Maintenance Playbook](./hsc-docs-maintenance-playbook.md)
- [Master Index](./99-master-index.md)

---

## 01-infra-hostinger

Função:
- substrate Hostinger do lado jogo + portal
- DNS/TLS da borda Hostinger
- Nginx público do portal e da Static API v2
- Docker host
- systemd e automação da v2

Arquivos principais:
- [Infra Hostinger — README](../01-infra-hostinger/README.md)
- [Infra Hostinger Architecture Runtime](../01-infra-hostinger/infra-hostinger-architecture-runtime.md)
- [Infra Hostinger Network, DNS and TLS](../01-infra-hostinger/infra-hostinger-network-dns-tls.md)
- [Nginx Static Serving](../01-infra-hostinger/nginx-static-serving.md)
- [Systemd Automation](../01-infra-hostinger/systemd-automation.md)
- [Infra Hostinger References Inventory](../01-infra-hostinger/infra-hostinger-references-inventory.md)

---

## 02-game-panel

Função:
- AMP / CS2 runtime
- instância `MixHAXIXE01`
- MatchZy
- plugins carregados
- camada operacional do servidor
- baseline administrativa do lado jogo

Arquivos principais:
- [Game Panel — README](../02-game-panel/README.md)
- [Game Panel Architecture Runtime](../02-game-panel/game-panel-architecture-runtime.md)
- [CS2 Server Configuration](../02-game-panel/cs2-server-configuration.md)
- [MariaDB Runtime](../02-game-panel/mariadb-runtime.md)
- [MatchZy](../02-game-panel/matchzy.md)
- [Plugins Installed](../02-game-panel/plugins-installed.md)
- [Game Panel Operational Runbooks](../02-game-panel/game-panel-operational-runbooks.md)
- [Game Panel Observability and Troubleshooting](../02-game-panel/game-panel-observability-troubleshooting.md)
- [Game Panel References Inventory](../02-game-panel/game-panel-references-inventory.md)
- [Game Panel Admin Authority Runtime](../02-game-panel/game-panel-admin-authority-runtime.md)
- [Game Panel Admin Operational Runbooks](../02-game-panel/game-panel-admin-operational-runbooks.md)
- [Admin Command Profiles](../02-game-panel/game-panel-admin-command-profiles.md)
- [Admin References Inventory](../02-game-panel/game-panel-admin-references-inventory.md)

---

## 03-portal-estatico

Função:
- Portal público
- Static API v2
- ETL Bash
- publishing Nginx
- mirror same-origin de conteúdo
- runbooks operacionais da camada pública

Arquivos principais:
- [Portal Estático — README](../03-portal-estatico/README.md)
- [Portal Estático Architecture Runtime](../03-portal-estatico/portal-estatico-architecture-runtime.md)
- [Data Sources — MatchZy SQLite](../03-portal-estatico/data-sources-matchzy-sqlite.md)
- [ETL Bash Pipeline](../03-portal-estatico/etl-bash-pipeline.md)
- [ETL Runtime Reconciliation](../03-portal-estatico/etl-runtime-reconciliation.md)
- [ETL Runtime Materialization Runbook](../03-portal-estatico/etl-runtime-materialization-runbook.md)
- [ETL Repository — Minimal Shell CI](../03-portal-estatico/etl-repository-minimal-shell-ci.md)
- [Portal Estático Frontend Structure](../03-portal-estatico/portal-estatico-frontend-structure.md)
- [JSON Contracts](../03-portal-estatico/json-contracts.md)
- [Nginx Publishing and Cache](../03-portal-estatico/nginx-publishing-cache.md)
- [Portal Estático Observability and Troubleshooting](../03-portal-estatico/portal-estatico-observability-troubleshooting.md)
- [Portal Estático Operational Runbooks](../03-portal-estatico/portal-estatico-operational-runbooks.md)
- [Portal Estático References Inventory](../03-portal-estatico/portal-estatico-references-inventory.md)
- [SQL Queries and Views](../03-portal-estatico/sql-queries-and-views.md)
- [Static API v2](../03-portal-estatico/static-api-v2.md)

---

## 04-infra-aws-lightsail

Função:
- runtime da Auth API
- reverse proxy Nginx
- Node.js via systemd
- MariaDB local
- deploy / rollback
- backup / restore
- camada dinâmica administrativa do ecossistema

Arquivos principais:
- [Infra AWS Lightsail — README](../04-infra-aws-lightsail/README.md)
- [Infra AWS Lightsail Architecture Runtime](../04-infra-aws-lightsail/infra-aws-lightsail-architecture-runtime.md)
- [Auth API Operations](../04-infra-aws-lightsail/auth-api-operations.md)
- [Backup / Restore](../04-infra-aws-lightsail/backup-restore.md)
- [Deploy / Release / Rollback](../04-infra-aws-lightsail/deploy-release-rollback.md)
- [Infra AWS Lightsail Network, DNS and TLS](../04-infra-aws-lightsail/infra-aws-lightsail-network-dns-tls.md)
- [Nginx Reverse Proxy](../04-infra-aws-lightsail/nginx-reverse-proxy.md)
- [Node Systemd](../04-infra-aws-lightsail/node-systemd.md)
- [Infra AWS Lightsail Observability and Troubleshooting](../04-infra-aws-lightsail/infra-aws-lightsail-observability-troubleshooting.md)
- [Infra AWS Lightsail References Inventory](../04-infra-aws-lightsail/infra-aws-lightsail-references-inventory.md)

---

## 05-backoffice-admin

Função:
- SPA administrativa do HSC
- shell administrativo
- auth frontend, guards e RBAC
- contratos admin com a Auth API
- runbooks funcionais do produto administrativo
- superfícies administrativas de `news`, `seasons` e `events`

Arquivos principais:
- [Backoffice Admin — README](../05-backoffice-admin/README.md)
- [Admin API Contracts](../05-backoffice-admin/admin-api-contracts.md)
- [News Admin API Contracts](../05-backoffice-admin/news-admin-api-contracts.md)
- [News Admin Integration and Evolution](../05-backoffice-admin/news-admin-integration-and-evolution.md)
- [News Admin Frontend Implementation Runtime](../05-backoffice-admin/news-admin-frontend-implementation-runtime.md)
- [News Functional Smoke Guide](../05-backoffice-admin/news-functional-smoke-guide.md)
- [Seasons Admin Functional Smoke Guide](../05-backoffice-admin/seasons-admin-list-functional-smoke-guide.md)
- [Backoffice UI Material Foundation](../05-backoffice-admin/backoffice-ui-material-foundation.md)
- [Auth, RBAC and Guards](../05-backoffice-admin/auth-rbac-and-guards.md)
- [Backoffice Admin Architecture Runtime](../05-backoffice-admin/backoffice-admin-architecture-runtime.md)
- [Backoffice Admin Frontend Structure](../05-backoffice-admin/backoffice-admin-frontend-structure.md)
- [Backoffice Admin Operational Runbooks](../05-backoffice-admin/backoffice-admin-operational-runbooks.md)
- [Backoffice Admin References Inventory](../05-backoffice-admin/backoffice-admin-references-inventory.md)

---

## Situação dos contextos canônicos

## 01 — Infra Hostinger

Status:
- **canônico**
- **reconciliado em pontos críticos**
- **operacionalmente confiável**

Leitura sintética:
- substrate do lado Hostinger estabilizado
- rede, publicação estática e automação já tratadas como baseline viva

---

## 02 — Game Panel

Status:
- **canônico**
- **reconciliado em pontos críticos**
- **operacionalmente confiável**

Principais fatos já reconciliados:
- a instância oficial do lado jogo continua sendo `MixHAXIXE01`
- MatchZy continua como peça central do runtime competitivo
- plugins relevantes do lado jogo permanecem inventariados
- a autoridade administrativa do lado jogo foi centralizada no CounterStrikeSharp
- `admins.json` + `admin_groups.json` passaram a ser a baseline canônica de admin do contexto
- `#hsc/root` é o grupo administrativo canônico ativo do checkpoint atual
- MatchZy e CS2-SimpleAdmin deixaram de manter autoridade paralela como fonte primária
- o resíduo legado de configuração do Railway foi removido do `CS2-SimpleAdmin.json`
- o contexto agora possui trilha documental própria para autoridade, manutenção, comandos e inventário administrativo

Leitura sintética:
- o lado jogo já não depende mais de memória informal para governança de admins
- a separação entre admin de servidor e admin de partida segue preservada por responsabilidade
- o contexto está mais sustentável para operação e evolução futura

---

## 03 — Portal Estático

Status:
- **canônico**
- **reconciliado em pontos críticos**
- **operacionalmente confiável**

Leitura sintética:
- cadeia MatchZy SQLite → ETL → Static API v2 → portal segue como trilha principal do lado público
- o runtime ETL da v2 já foi reconciliado com o repositório `hsc-cs2-etl`
- a materialização em `/usr/local/bin` já passou a ser tratada como camada runtime, e não como superfície manual de autoria

---

## 04 — Infra AWS Lightsail

Status:
- **canônico**
- **reconciliado em pontos críticos**
- **operacionalmente confiável**

Leitura sintética:
- Auth API e infraestrutura dinâmica seguem com baseline operacional viva própria

---

## 05 — Backoffice Admin

Status:
- **canônico**
- **ativo**
- **em expansão incremental de domínio**

Leitura sintética:
- shell administrativo, auth frontend e contratos iniciais já existem
- News já possui ciclo PROD validado atravessando Backoffice Admin, Auth API, ETL Hostinger e Portal Estático
- `GET /admin/news/:id` já é contrato admin real para edição com `content`
- Seasons já possui listagem administrativa em `/seasons`
- Seasons já possui criação administrativa em `draft` por `/seasons/new`
- Seasons já possui edição administrativa de metadados por `/seasons/:slug/edit`
- Seasons já possui ações administrativas de lifecycle em `/seasons` para ativar e fechar
- `GET /admin/seasons`, `GET /admin/seasons/:slug`, `POST /admin/seasons`, `PATCH /admin/seasons/:slug`, `POST /admin/seasons/:slug/activate` e `POST /admin/seasons/:slug/close` já são contratos admin reais consumidos pelo Backoffice
- `cover_image_url` de Seasons já é suportado pela Auth API, pelo Backoffice, pela fonte ETL da Static API v2 e pelo código-fonte do Portal Angular
- no Portal Angular, `SeasonDto` já tipa `cover_image_url`, `seasonCoverImage(...)` já prioriza esse campo, cards/heróis de Seasons e Ranking já usam `--season-cover`, e `npm run build` passou
- Backoffice já possui fundação UI Material para feedback transitório, feedback persistente, confirmação e input simples em fluxos principais de `news`, `seasons` e `users`
- dark mode, toggle de tema, design system completo e padronização visual total permanecem como lacunas futuras
- ranking, partidas por season, materialização runtime/prod do ETL atualizado, validação pública real da Static API v2 e validação visual pública do Portal com dados reais seguem como lacunas futuras de Seasons/Admin
- a tendência agora é expansão por features e superfícies de produto sem perder reconciliação runtime

---

## Regra final de leitura

Quando um leitor quiser entender um tema do HSC, a ordem correta é:

1. entrar pelo `README.md` do contexto
2. ler os documentos especializados daquele contexto
3. voltar a este índice apenas para navegação transversal

Este índice existe para coordenar.  
A verdade continua vivendo nos contextos canônicos e em seus documentos especializados.
