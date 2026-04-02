# References and Inventory

## Navegação rápida

- [Home da documentação](../README.md)
- [Portal Estático](./README.md)
- [Master Index](../00-governance/99-master-index.md)

---
## Objetivo

Consolidar o inventário de referências, evidências, artefatos operacionais, paths críticos e itens pendentes de validação do contexto Portal Estático do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- quais documentos alimentam este contexto
- quais artefatos reais do host e da publicação pública são source of truth operacional
- quais serviços, paths e componentes são críticos para o portal e para a Static API v2
- quais comandos ajudam a validar rapidamente a camada pública
- quais pontos ainda dependem de confirmação direta no ambiente real
- quais são os limites documentais deste contexto

---

## Escopo

Este documento cobre:

- documentos de origem do contexto
- artefatos reais da Static API v2 e do portal
- serviços principais da camada pública
- hostnames e paths públicos conhecidos
- paths críticos no filesystem
- comandos de validação
- itens pendentes de confirmação
- limites documentais do contexto

Este documento não cobre em profundidade:

- infraestrutura base completa da Hostinger
- operação detalhada do servidor CS2
- Auth API no Lightsail
- credenciais e arquivos sensíveis
- histórico completo de mudanças

Esses assuntos vivem nos documentos especializados de outros contextos, bem como em impl-logs e material legado.

---

## Estado atual

O contexto `03-portal-estatico` já possui estrutura canônica definida para documentar:

- arquitetura de runtime do portal
- Static API v2
- fonte SQLite do MatchZy
- pipeline ETL Bash
- camada SQL versionada
- contratos JSON públicos
- estrutura do frontend estático
- publishing e cache via Nginx
- runbooks operacionais
- observabilidade e troubleshooting da camada pública

Neste estágio da reconciliação, o contexto já possui confirmação operacional suficiente para fixar sem ambiguidade:

- os hostnames públicos canônicos:
  - `haxixesmokeclub.com`
  - `www.haxixesmokeclub.com`
- o path público oficial do portal:
  - `/portal/cs2/`
- o path público oficial da Static API v2:
  - `/api/cs2/v2/`
- o path público same-origin do mirror de News:
  - `/content/news/`
- a árvore pública do portal:
  - `/var/www/portal/cs2/`
- a árvore pública da Static API v2:
  - `/var/www/api/cs2/v2/`
- o path canônico vivo do `matchzy.db`
- a desativação explícita de `/api/cs2/v1/`

Ainda restam pendências menores, principalmente ligadas a:

- expansão controlada do inventário de materialização e release do runtime ETL
- formalização incremental de variáveis operacionais do ETL além do baseline já parametrizado
- evolução futura do contrato de deploy sem quebrar o `systemd` atual
- formalização do gate mínimo de CI do repositório ETL

---

## Source of truth / evidências

As evidências que sustentam este contexto se dividem em quatro grupos principais.

### 1. Documentação consolidada do ecossistema

Usada para:

- visão macro do papel do Portal Estático no HSC
- separação entre Hostinger, Game Panel, Portal Estático e Lightsail
- topologia geral da camada pública

### 2. Documentação específica do portal

Usada para:

- Static API v2
- ETL Bash
- frontend estático
- Nginx publishing
- mirror same-origin de conteúdo
- runbooks da camada pública

### 3. Impl-logs e registros incrementais

Usados para:

- rastrear mudanças em publishing, assets, paths, cache e mirror
- registrar ajustes relevantes da camada pública
- preservar rastreabilidade da evolução do portal

### 4. Ambiente real da Hostinger

Usado para:

- validar hostnames públicos canônicos
- validar os paths HTTP públicos realmente ativos
- validar os blocos `location`, `root` e `alias`
- validar o path canônico do `matchzy.db`
- validar a estrutura real publicada em `/var/www/portal/cs2/` e `/var/www/api/cs2/v2/`

Regra principal:

- quando houver conflito entre documento histórico e ambiente real validado, prevalece o ambiente real validado

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/portal-estatico-architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/sql-queries-and-views.md`
- `docs/03-portal-estatico/json-contracts.md`
- `docs/03-portal-estatico/portal-estatico-frontend-structure.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/03-portal-estatico/portal-estatico-operational-runbooks.md`
- `docs/03-portal-estatico/portal-estatico-observability-troubleshooting.md`
- `docs/03-portal-estatico/etl-runtime-reconciliation.md`
- `docs/03-portal-estatico/etl-runtime-materialization-runbook.md`
- `docs/03-portal-estatico/etl-repository-minimal-shell-ci.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/infra-hostinger-network-dns-tls.md`

Este documento não substitui nenhum dos arquivos acima.  
Ele funciona como fechamento de inventário e referência do contexto.

---

## Documentos de origem

Os documentos abaixo são as principais fontes de extração e reconciliação deste contexto.

### Fontes primárias

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação específica do Portal Estático
- documentação específica da Static API v2
- documentação da CI mínima do repositório ETL

### Fontes secundárias

- documentação reconciliada da Infra Hostinger
- documentação reconciliada do Game Panel
- impl-logs ligados a publishing, cache, assets, mirror de conteúdo e ETL

### Fontes de apoio histórico

- master antigo
- blueprint histórico
- documentação legada do portal
- materiais `_old` úteis para reconciliar paths, convenções de API, publishing e evolução da v2

Regra documental:

- fontes históricas ajudam a reconciliar
- mas não governam o canônico sozinhas

---

## Artefatos reais do contexto

Os artefatos reais conhecidos ou esperados deste contexto incluem:

### Hostnames públicos canônicos

- `haxixesmokeclub.com`
- `www.haxixesmokeclub.com`

### Hostnames não ativos para esta borda no estado atual

- `portal.haxixesmokeclub.com`
- `api.haxixesmokeclub.com`

### Publicação pública do portal

- `/portal/cs2/`

### Publicação pública dos assets do portal

- `/portal/cs2/assets/`

### Publicação pública da Static API v2

- `/api/cs2/v2/`

### Mirror same-origin de News

- `/content/news/`

### Árvore pública do portal

- `/var/www/portal/cs2/`

### Árvore pública da Static API v2

- `/var/www/api/cs2/v2/`

### Base operacional do portal / ETL

- `/opt/cs2-portal/`
- base de `locks`, `state`, `sql` e scripts `content/news` no contrato atual

### Lock global conhecido

- `/opt/cs2-portal/locks/gen-all-v2.lock`

### SQLs versionados

- `/opt/cs2-portal/sql/`

### Scripts operacionais

- `/usr/local/bin/`
- camada runtime materializada do ETL


### Repositório versionado do runtime ETL

- `14vosx/hsc-cs2-etl`

### Tag operacional reconciliada

- `etl-v0.3.0`

### Evidência bruta da baseline importada

- `_reconcile/2026-04-01/vps-runtime/raw/` dentro do repositório ETL

### Script de materialização do runtime

- `scripts/materialize-etl-runtime.sh` no repositório ETL

### Workflow mínimo de CI do repositório ETL

- `.github/workflows/ci-shell.yml` no repositório ETL
- trigger em `push` e `pull_request`
- cobertura explícita de `bin/*.sh` e `scripts/*.sh`
- `bash -n` + `shellcheck -S warning`

### Fonte de dados principal

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db`

### Backup identificado da fonte

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/backup-hsc/MatchZy.live-dir.pre-upstream.20260317-205843/matchzy.db`

Esses artefatos devem ser tratados como inventário-base do contexto até revisão explícita.

---

## Componentes estruturais relevantes

Os componentes estruturais mais importantes já reconciliados neste contexto são:

### Portal estático

Papel:
- frontend público do ecossistema
- consumo da Static API v2
- navegação SPA sob `/portal/cs2/`

### Static API v2

Papel:
- camada pública oficial de dados JSON
- fonte consumida pelo frontend
- substituta oficial da antiga v1

### ETL Bash

Papel:
- gerar artefatos públicos a partir da fonte SQLite
- atualizar a árvore pública da v2

### GitHub Actions do repositório ETL

Papel:
- aplicar gate mínimo de shell antes do merge
- reduzir regressão boba na camada versionada do ETL

### Nginx da Hostinger

Papel:
- publicar portal, assets, v2 e mirror de conteúdo
- aplicar cache e regras de compatibilidade

### SQLite do MatchZy

Papel:
- fonte principal da v2
- origem estatística do portal

### Mirror same-origin de conteúdo

Papel:
- publicar conteúdo JSON sob o mesmo domínio do portal
- reduzir atrito de consumo pelo frontend

---

## Componentes já reconciliados

Os itens abaixo já possuem relevância reconciliada suficiente no contexto atual.

### Hostnames canônicos da camada pública

- `haxixesmokeclub.com`
- `www.haxixesmokeclub.com`

### Prefixo oficial do portal

- `/portal/cs2/`

### Prefixo oficial da Static API v2

- `/api/cs2/v2/`

### Prefixo oficial do mirror de News

- `/content/news/`

### Fonte principal da v2

- `matchzy.db`

### Tabelas estruturais da fonte

- `matchzy_stats_matches`
- `matchzy_stats_maps`
- `matchzy_stats_players`

### Desativação explícita da v1

- `/api/cs2/v1/` retorna `404`

### Compatibilidades públicas ativas

- `/api/ranking.json` → `/api/cs2/v2/ranking.json`
- `/api/matches.json` → `/api/cs2/v2/matches.json`
- `/api/health.json` → `/api/cs2/v2/health.json`
- `/api/steam/<id>.json` → `/api/cs2/v2/player/<id>.json`
- `/portal/` → `/portal/cs2/`
- `/portal/ranking/` → `/portal/cs2/ranking/`
- `/portal/matches/` → `/portal/cs2/matches/`

Esses itens já devem ser tratados como parte da verdade operacional do contexto.

---

## Paths críticos

Os paths críticos conhecidos deste contexto incluem:

### Portal público

- `/var/www/portal/cs2/`

### Assets públicos do portal

- `/var/www/portal/cs2/assets/`

### Static API v2 pública

- `/var/www/api/cs2/v2/`

### Base operacional do portal

- `/opt/cs2-portal/`

### Locks do ETL

- `/opt/cs2-portal/locks/`

### Statefiles do ETL

- `/opt/cs2-portal/state/`

### SQLs versionados

- `/opt/cs2-portal/sql/`

### Scripts operacionais

- `/usr/local/bin/`
- camada runtime materializada do ETL

### Fonte viva do `matchzy.db`

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db`

### Backup identificado do `matchzy.db`

- `/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/backup-hsc/MatchZy.live-dir.pre-upstream.20260317-205843/matchzy.db`

Regra prática:

- o path em `addons/counterstrikesharp/plugins/MatchZy/` é a fonte viva canônica
- o path em `backup-hsc/` é backup histórico e não deve ser tratado como fonte ativa
- qualquer mudança nesses paths deve ser tratada como alteração relevante e refletida no canônico e, quando necessário, no impl-log

---

## Regras públicas de publicação já reconciliadas

As regras públicas mais relevantes já reconciliadas no runtime incluem:

### Assets estáveis do portal

- publicados em `/portal/cs2/assets/`
- `alias /var/www/portal/cs2/assets/`
- `expires 1h`
- `Cache-Control: public, max-age=3600`

### Assets legados desativados

- `/portal/cs2/assets/releases/` → `410`
- `/portal/cs2/assets/current/` → `410`

### Static API v2

- publicada em `/api/cs2/v2/`
- `alias /var/www/api/cs2/v2/`
- `default_type application/json`
- `Cache-Control: no-store, no-cache, max-age=0, must-revalidate`
- `Pragma: no-cache`
- `Expires: 0`
- `X-HSC-API: v2`

### Mirror de News

- publicado em `/content/news/`
- servido a partir da base `/var/www/api/cs2/v2/`
- política de `Cache-Control: no-store, max-age=0`

### Neutralização de Service Worker

- `/ServiceWorker.js`
- alias para `/var/www/portal/sw-kill-ServiceWorker.js`
- `Cache-Control: no-store, max-age=0`

---

## Dependências cruzadas

Os principais workloads e dependências cruzadas deste contexto incluem:

### Dependência da Infra Hostinger

O Portal Estático depende de:

- host Debian
- Nginx
- filesystem público
- systemd/timers do host
- permissões corretas

### Dependência do Game Panel

O Portal Estático depende do Game Panel para:

- origem do `matchzy.db`
- continuidade da produção de stats
- estabilidade do schema e do path da persistência local

### Dependência do ETL

A borda pública depende de:

- geração correta dos arquivos JSON
- publicação correta na árvore pública
- ausência de artefato parcial ou stale

### Dependência do frontend

O frontend depende de:

- paths públicos corretos
- contratos JSON coerentes
- borda pública previsível
- cache compatível com a cadência de atualização

---

## Comandos de validação

Os comandos abaixo formam um kit mínimo de validação da camada pública.

### Validar portal público

```bash
curl -I https://haxixesmokeclub.com/portal/cs2/
curl -I https://www.haxixesmokeclub.com/portal/cs2/
```

### Validar health da v2

```bash
curl -I https://haxixesmokeclub.com/api/cs2/v2/health.json
curl -sS https://haxixesmokeclub.com/api/cs2/v2/health.json
```

### Validar ranking da v2

```bash
curl -I https://haxixesmokeclub.com/api/cs2/v2/ranking.json
curl -sS https://haxixesmokeclub.com/api/cs2/v2/ranking.json
```

### Validar mirror same-origin de News

```bash
curl -I https://haxixesmokeclub.com/content/news/
curl -sS https://haxixesmokeclub.com/content/news/
```

### Validar redirects de compatibilidade

```bash
curl -I https://haxixesmokeclub.com/api/health.json
curl -I https://haxixesmokeclub.com/api/matches.json
curl -I https://haxixesmokeclub.com/api/ranking.json
```

### Validar desativação da v1

```bash
curl -I https://haxixesmokeclub.com/api/cs2/v1/
```

### Validar sintaxe do Nginx no host

```bash
sudo nginx -t
```

### Validar configuração relevante do Nginx

```bash
sudo nginx -T | grep -nE "server_name|root |alias "
sudo nginx -T | sed -n '200,320p'
```

### Validar presença da fonte ativa

```bash
ls -l /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db ".tables"
```

---

## Itens pendentes de confirmação

Os itens abaixo ainda devem ser confirmados diretamente no ambiente real para elevar o grau de confiança do contexto.

### 1. Nomes exatos de services e timers do host

A intenção operacional já está reconciliada, mas ainda falta congelar os nomes exatos de todas as units relevantes da geração da v2 e do health.

### 2. Comando agregador real da regeneração da v2

A ordem normativa da pipeline já está correta, mas ainda falta fixar sem ambiguidade qual é o comando agregador real usado no runtime atual.

### 3. Inventário final dos scripts `gen-*` ativos

A família de scripts já está conceitualmente reconciliada, mas a fotografia final dos scripts realmente ativos em produção ainda pode ser refinada.

### 4. Root/alias final completo de todos os blocos públicos relevantes

Os blocos principais já estão reconciliados, mas ainda pode haver ganho em congelar a fotografia final completa do vhost para troubleshooting futuro.

### 5. Eventuais statefiles específicos que mereçam citação formal

A base `state/` já está posicionada corretamente, mas ainda pode ser útil formalizar statefiles críticos após validação adicional do host.

---

## Itens fora do escopo deste contexto

Os itens abaixo não pertencem ao inventário canônico do Portal Estático:

- runtime da Auth API no Lightsail
- MariaDB local da Auth API
- operação detalhada do servidor CS2
- AMP
- MatchZy como workflow competitivo completo
- DNS/TLS detalhado do Lightsail
- credenciais reais
- arquivos de acesso sensíveis
- documentação histórica não reconciliada

Esses itens pertencem a outros contextos ou devem permanecer fora do fluxo normal do repositório documental.

---

## Limites documentais do contexto

Os limites documentais deste contexto são:

- ele documenta a camada pública estática do ecossistema
- ele não documenta sozinho o substrate da Hostinger
- ele não documenta sozinho o lado jogo
- ele não substitui o índice mestre
- ele não substitui impl-logs históricos
- ele depende de confirmação periódica contra o ambiente real para permanecer confiável

---

## Critério de manutenção

Este documento deve ser atualizado quando houver:

- mudança de hostname público canônico
- mudança do prefixo público do portal
- mudança do prefixo público da v2
- mudança de path estrutural do `matchzy.db`
- mudança relevante na política de cache
- mudança de redirects de compatibilidade
- confirmação ou resolução de item pendente listado aqui

Mudanças pequenas de comportamento funcional devem ser refletidas primeiro no documento especializado correspondente, e não necessariamente aqui.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os artefatos reais da borda pública estiverem confirmados sem ambiguidade
- os hostnames canônicos, os prefixes públicos e os paths de filesystem estiverem claramente reconciliados
- o path vivo do `matchzy.db` estiver fixado
- os itens pendentes estiverem resolvidos ou explicitamente mantidos como pendência consciente
- ele puder ser usado como inventário confiável do Portal Estático sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / references and inventory
- Última revisão: 2026-03-18