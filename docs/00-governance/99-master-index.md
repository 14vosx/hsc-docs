# HSC Canonical Master Index

## Objetivo

Este documento é o índice mestre canônico do repositório documental do ecossistema HSC.

Ele existe para:

- apontar a estrutura oficial da documentação viva
- registrar o estado atual de maturidade dos contextos canônicos
- resumir os principais fatos operacionais já reconciliados no runtime real
- indicar quais pendências ainda existem de forma explícita
- evitar retorno ao modelo anterior de “master monolítico” como fonte primária de verdade

Regra canônica:

**a verdade documental do sistema vive por contexto, e não em um documento único gigante.**  
Este índice coordena a leitura. Ele não substitui os documentos especializados.

---

## Estado atual do repositório

A base documental canônica do HSC já está estruturada e utilizável.

Neste checkpoint, os quatro contextos principais já foram reconciliados com o runtime real em pontos operacionais de maior valor:

- Infra Hostinger
- Game Panel / CS2
- Portal Estático / Static API v2
- Infra AWS Lightsail / Auth API

Leitura correta do estado atual:

- o repositório já pode ser tratado como **base canônica viva**
- a maior parte das pendências críticas de realidade operacional já foi fechada
- o que resta agora é principalmente **acabamento fino, cleanup e validação adicional de restore**

---

## Estrutura canônica

## 00-governance

Função:
- governança documental
- regras do sistema documental
- índice mestre canônico

Arquivos principais:
- `docs/00-governance/README.md`
- `docs/00-governance/documentation-system.md`
- `docs/00-governance/99-master-index.md`

---

## 01-infra-hostinger

Função:
- substrate Hostinger do lado jogo + portal
- DNS/TLS da borda Hostinger
- Nginx público do portal e da Static API v2
- Docker host
- systemd e automação da v2

Arquivos principais:
- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/architecture-runtime.md`
- `docs/01-infra-hostinger/network-dns-tls.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/systemd-automation.md`
- `docs/01-infra-hostinger/references-inventory.md`

---

## 02-game-panel

Função:
- AMP / CS2 runtime
- instância `MixHAXIXE01`
- MatchZy
- plugins carregados
- camada operacional do servidor

Arquivos principais:
- `docs/02-game-panel/README.md`
- `docs/02-game-panel/architecture-runtime.md`
- `docs/02-game-panel/instance-mixhaxixe01.md`
- `docs/02-game-panel/matchzy.md`
- `docs/02-game-panel/plugins-installed.md`
- `docs/02-game-panel/references-inventory.md`

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
- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/03-portal-estatico/operational-runbooks.md`
- `docs/03-portal-estatico/references-inventory.md`

---

## 04-infra-aws-lightsail

Função:
- runtime da Auth API
- reverse proxy Nginx
- Node.js via systemd
- MariaDB local
- deploy / rollback
- backup / restore

Arquivos principais:
- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/architecture-runtime.md`
- `docs/04-infra-aws-lightsail/nginx-reverse-proxy.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/backup-restore.md`
- `docs/04-infra-aws-lightsail/references-inventory.md`

---

## 95-impl-log

Função:
- registrar mudanças relevantes
- preservar rastreabilidade das reconciliações
- servir de trilha histórica curta e formal

Arquivo atual:
- `docs/95-impl-log/2026-03-18-initial-canonical-context-migration.md`

---

## 97-audit

Função:
- registrar pendências reais pós-migração
- separar “falta documental” de “pendência operacional real”

Arquivo atual:
- `docs/97-audit/2026-03-18-post-migration-open-items.md`

---

## 98-legacy

Função:
- preservar masters e documentos históricos
- evitar que material legado continue governando o sistema novo

Arquivos principais:
- `docs/98-legacy/README.md`
- `docs/98-legacy/HSC_MASTER_DOCUMENTATION.md`
- `docs/98-legacy/HSC_Master_Blueprint.md`
- `docs/98-legacy/master-documents-migration-map.md`

---

## Situação dos contextos canônicos

## 01 — Infra Hostinger

Status:
- **canônico**
- **reconciliado em pontos críticos**
- **operacionalmente confiável**

Principais fatos já reconciliados:
- hostname técnico real:
  - `srv1353392.hstgr.cloud`
- hostnames públicos canônicos reais:
  - `haxixesmokeclub.com`
  - `www.haxixesmokeclub.com`
- hostnames não ativos para a borda Hostinger:
  - `portal.haxixesmokeclub.com`
  - `api.haxixesmokeclub.com`
- arquivo principal real do Nginx:
  - `/etc/nginx/conf.d/srv1353392.hstgr.cloud.conf`
- heartbeat real da automação da v2:
  - `gen-all-v2.timer`
  - `gen-health.timer`
- agregador real da v2:
  - `/usr/local/bin/gen-all-v2.sh`
- base operacional real:
  - `/opt/cs2-portal/`

---

## 02 — Game Panel

Status:
- **canônico**
- **reconciliado em runtime**
- **operacionalmente confiável**

Principais fatos já reconciliados:
- instância oficial:
  - `MixHAXIXE01`
- container relevante:
  - `AMP_MixHAXIXE01`
- MatchZy real:
  - `0.8.15`
- path vivo real do banco:
  - `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db`
- inventário real de plugins carregados:
  - `WeaponPaints`
  - `PlayerSettings [Core]`
  - `CS2-SimpleAdmin Fun Commands`
  - `MatchZy`
  - `MenuManager [Core]`
  - `[CS2-SimpleAdmin] Stealth Module`
  - `CS2-SimpleAdmin (RELEASE)`

---

## 03 — Portal Estático

Status:
- **canônico**
- **reconciliado em paths públicos e pipeline**
- **operacionalmente confiável**

Principais fatos já reconciliados:
- path público oficial do portal:
  - `/portal/cs2/`
- path público oficial da Static API v2:
  - `/api/cs2/v2/`
- mirror same-origin de News:
  - `/content/news/`
- árvore pública do portal:
  - `/var/www/portal/cs2/`
- árvore pública da v2:
  - `/var/www/api/cs2/v2/`
- pipeline real da v2:
  - `/usr/local/bin/gen-all-v2.sh`
- ordem real reconciliada da pipeline:
  - `gen-matches.sh`
  - `gen-ranking.sh`
  - `gen-players-incremental.sh`
  - `gen-players-from-ranking.sh`
  - `gen-maps.sh`
- v1 pública desativada:
  - `/api/cs2/v1/` retorna `404`

---

## 04 — Infra AWS Lightsail

Status:
- **canônico**
- **reconciliado em runtime real**
- **operacionalmente confiável**

Principais fatos já reconciliados:
- hostname público canônico:
  - `auth-api.haxixesmokeclub.com`
- unit real da app:
  - `hsc-auth-api.service`
- usuário real:
  - `hscadmin`
- working directory real:
  - `/opt/hsc/hsc-auth-api`
- `ExecStart` real:
  - `/usr/bin/node index.js`
- binding observado:
  - `0.0.0.0:3000`
- vhost real:
  - `/etc/nginx/sites-available/hsc-auth-api`
- script real de backup:
  - `/opt/hsc/backup-mariadb.sh`
- diretório real dos dumps:
  - `/opt/hsc/backups/mariadb/`
- log real:
  - `/opt/hsc/backups/mariadb/backup.log`
- invocador real do backup:
  - crontab do root
- linha real do cron:
  - `15 3 * * * /opt/hsc/backup-mariadb.sh`
- timezone do host:
  - `Etc/UTC`
- retenção real configurada:
  - `14 dias`

---

## Pendências reais que ainda permanecem

As pendências restantes, neste checkpoint, já não são blocos estruturais grandes.

## Pendências de maior valor ainda abertas

### 1. Restore validado ponta a ponta no Lightsail

Falta ainda:
- executar e registrar um restore real controlado
- idealmente em base temporária
- com validação pós-restore

Esta é hoje a pendência operacional mais relevante ainda não fechada de forma prática.

### 2. Cleanup do drift residual da Auth API antiga na Hostinger

Ainda existe evidência residual no host Hostinger de:
- vhost/config antiga de `auth-api.haxixesmokeclub.com`
- `hsc-auth-api.service` residual

Isto já está identificado, mas o cleanup técnico ainda não foi executado/documentado como concluído.

### 3. Fotografia final completa do vhost Nginx da Hostinger

Os blocos críticos já foram reconciliados.  
Ainda pode haver ganho em congelar uma fotografia final mais completa do vhost para auditoria futura.

### 4. Eventual estratégia off-host de backup da Auth API

O backup local já está reconciliado.  
Ainda não foi confirmado, neste ciclo, se existe cópia externa/off-host dos dumps.

---

## O que já pode ser considerado fechado neste checkpoint

Neste checkpoint, já podem ser tratados como fechados:

- separação canônica dos quatro contextos
- hostnames reais do lado Hostinger
- hostnames e runtime real do Lightsail
- path vivo real do `matchzy.db`
- inventário real de plugins carregados
- pipeline real da Static API v2
- heartbeat real da v2 no host
- reverse proxy real da Auth API
- unit real do Node via `systemd`
- diretório real dos dumps do MariaDB
- cron real do backup
- retenção real de backup

Leitura correta:

**o sistema documental já passou da fase de “migração estrutural” e entrou em fase de “refino operacional incremental”.**

---

## Ordem recomendada de leitura

Para visão macro do sistema:

1. `docs/00-governance/README.md`
2. `docs/00-governance/documentation-system.md`
3. `docs/00-governance/99-master-index.md`

Para lado Hostinger + Portal:

1. `docs/01-infra-hostinger/README.md`
2. `docs/01-infra-hostinger/references-inventory.md`
3. `docs/03-portal-estatico/README.md`
4. `docs/03-portal-estatico/references-inventory.md`
5. `docs/03-portal-estatico/operational-runbooks.md`

Para lado jogo:

1. `docs/02-game-panel/README.md`
2. `docs/02-game-panel/references-inventory.md`
3. `docs/02-game-panel/plugins-installed.md`
4. `docs/02-game-panel/matchzy.md`

Para lado Auth API:

1. `docs/04-infra-aws-lightsail/README.md`
2. `docs/04-infra-aws-lightsail/references-inventory.md`
3. `docs/04-infra-aws-lightsail/node-systemd.md`
4. `docs/04-infra-aws-lightsail/backup-restore.md`
5. `docs/04-infra-aws-lightsail/auth-api-operations.md`

---

## Regra editorial do índice mestre

Este arquivo deve ser atualizado quando houver:

- mudança de status de um contexto canônico
- fechamento ou abertura de pendência operacional relevante
- alteração do papel estrutural de algum contexto
- reconciliação importante de runtime que mude a leitura macro do sistema
- mudança de ordem recomendada de leitura

Mudanças pequenas e locais devem ser registradas primeiro nos documentos especializados.

---

## Declaração de checkpoint

### Checkpoint atual

**Checkpoint válido e forte.**

Motivos:

- principais verdades operacionais do sistema já foram reconciliadas
- documentação já está segmentada por contexto e utilizável
- principais riscos de drift documental já foram reduzidos
- o que resta agora é majoritariamente refino, cleanup e validação adicional

Leitura correta deste checkpoint:

- **não é o fim do trabalho documental**
- mas **já é um ponto seguro, auditável e confiável para pausar**

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: governança / índice mestre
- Última revisão: 2026-03-18