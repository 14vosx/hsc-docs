# HSC Docs — Home

## Navegação rápida

- [Governance](./00-governance/README.md)
- [Master Index](./00-governance/99-master-index.md)
- [Infra Hostinger](./01-infra-hostinger/README.md)
- [Game Panel](./02-game-panel/README.md)
- [Portal Estático](./03-portal-estatico/README.md)
- [Infra AWS Lightsail](./04-infra-aws-lightsail/README.md)
- [Backoffice Admin](./05-backoffice-admin/README.md)
- [Impl Log](./95-impl-log/README.md)
- [Audit](./97-audit/README.md)
- [Legacy](./98-legacy/README.md)

---
## Objetivo

Este é o ponto de entrada principal da documentação viva do ecossistema HSC.

A navegação recomendada do repositório começa aqui e segue por contexto canônico.

---

## Entrada principal

- [Governance](./00-governance/README.md)
- [Documentation System](./00-governance/documentation-system.md)
- [Master Index](./00-governance/99-master-index.md)

---

## Contextos canônicos

- [01 — Infra Hostinger](./01-infra-hostinger/README.md)
- [02 — Game Panel](./02-game-panel/README.md)
- [03 — Portal Estático](./03-portal-estatico/README.md)
- [04 — Infra AWS Lightsail](./04-infra-aws-lightsail/README.md)
- [05 — Backoffice Admin](./05-backoffice-admin/README.md)

---

## Trilha operacional por domínio

### Infraestrutura base
- [Infra Hostinger](./01-infra-hostinger/README.md)
- [Architecture Runtime — Hostinger](./01-infra-hostinger/infra-hostinger-architecture-runtime.md)
- [Network, DNS and TLS](./01-infra-hostinger/infra-hostinger-network-dns-tls.md)
- [Nginx Static Serving](./01-infra-hostinger/nginx-static-serving.md)
- [Systemd Automation](./01-infra-hostinger/systemd-automation.md)

### Servidor CS2 / Game Panel
- [Game Panel](./02-game-panel/README.md)
- [AMP Instance Manager](./02-game-panel/amp-instance-manager.md)
- [CS2 Server Configuration](./02-game-panel/cs2-server-configuration.md)
- [Instance MixHAXIXE01](./02-game-panel/instance-mixhaxixe01.md)
- [MatchZy](./02-game-panel/matchzy.md)
- [Plugins Installed](./02-game-panel/plugins-installed.md)

### Portal público e Static API v2
- [Portal Estático](./03-portal-estatico/README.md)
- [Static API v2](./03-portal-estatico/static-api-v2.md)
- [ETL Bash Pipeline](./03-portal-estatico/etl-bash-pipeline.md)
- [ETL Runtime Reconciliation](./03-portal-estatico/etl-runtime-reconciliation.md)
- [ETL Runtime Materialization Runbook](./03-portal-estatico/etl-runtime-materialization-runbook.md)
- [ETL Repository — Minimal Shell CI](./03-portal-estatico/etl-repository-minimal-shell-ci.md)
- [Data Sources — MatchZy SQLite](./03-portal-estatico/data-sources-matchzy-sqlite.md)
- [JSON Contracts](./03-portal-estatico/json-contracts.md)
- [Frontend Structure](./03-portal-estatico/portal-estatico-frontend-structure.md)
- [Nginx Publishing and Cache](./03-portal-estatico/nginx-publishing-cache.md)

### Auth API / camada dinâmica
- [Infra AWS Lightsail](./04-infra-aws-lightsail/README.md)
- [Architecture Runtime — Lightsail](./04-infra-aws-lightsail/infra-aws-lightsail-architecture-runtime.md)
- [Nginx Reverse Proxy](./04-infra-aws-lightsail/nginx-reverse-proxy.md)
- [Node Systemd](./04-infra-aws-lightsail/node-systemd.md)
- [MariaDB Local](./04-infra-aws-lightsail/mariadb-local.md)
- [Auth API Operations](./04-infra-aws-lightsail/auth-api-operations.md)
- [Deploy / Release / Rollback](./04-infra-aws-lightsail/deploy-release-rollback.md)

### Backoffice administrativo
- [Backoffice Admin](./05-backoffice-admin/README.md)
- [Architecture Runtime — Backoffice](./05-backoffice-admin/backoffice-admin-architecture-runtime.md)
- [Frontend Structure — Backoffice](./05-backoffice-admin/backoffice-admin-frontend-structure.md)
- [Auth, RBAC and Guards](./05-backoffice-admin/auth-rbac-and-guards.md)
- [Admin API Contracts](./05-backoffice-admin/admin-api-contracts.md)
- [Operational Runbooks — Backoffice](./05-backoffice-admin/backoffice-admin-operational-runbooks.md)

---

## Logs, auditoria e legado

- [Impl Log](./95-impl-log/README.md)
- [Audit](./97-audit/README.md)
- [Legacy](./98-legacy/README.md)

---

## Fluxos principais

### Fluxo jogo → dados → portal
- [MatchZy](./02-game-panel/matchzy.md)
- [Data Sources — MatchZy SQLite](./03-portal-estatico/data-sources-matchzy-sqlite.md)
- [SQL Queries and Views](./03-portal-estatico/sql-queries-and-views.md)
- [ETL Bash Pipeline](./03-portal-estatico/etl-bash-pipeline.md)
- [ETL Runtime Reconciliation](./03-portal-estatico/etl-runtime-reconciliation.md)
- [ETL Runtime Materialization Runbook](./03-portal-estatico/etl-runtime-materialization-runbook.md)
- [ETL Repository — Minimal Shell CI](./03-portal-estatico/etl-repository-minimal-shell-ci.md)
- [Static API v2](./03-portal-estatico/static-api-v2.md)
- [Frontend Structure](./03-portal-estatico/portal-estatico-frontend-structure.md)

### Fluxo admin → auth → backoffice
- [Backoffice Admin](./05-backoffice-admin/README.md)
- [Auth, RBAC and Guards](./05-backoffice-admin/auth-rbac-and-guards.md)
- [Admin API Contracts](./05-backoffice-admin/admin-api-contracts.md)
- [Auth API Operations](./04-infra-aws-lightsail/auth-api-operations.md)
- [Nginx Reverse Proxy](./04-infra-aws-lightsail/nginx-reverse-proxy.md)

---

## Regra de navegação

Sempre que possível:

1. comece por esta Home
2. entre no contexto canônico correto
3. siga para o documento especializado
4. evite usar os arquivos legados como ponto inicial de leitura