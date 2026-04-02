# ETL Bash Pipeline

## Objetivo

Documentar a pipeline ETL Bash real do contexto Portal Estático do ecossistema HSC, registrando o orquestrador canônico reconciliado no host, os scripts que compõem a geração da Static API v2, a base operacional usada pela automação e os pontos de acoplamento entre fonte, geração e publicação pública.

Este documento existe para registrar, de forma estável e auditável:

- qual é o comando agregador real da v2 no runtime atual
- quais scripts compõem a pipeline reconciliada
- em que ordem a pipeline roda
- quais locks e statefiles participam da operação
- como a base `/opt/cs2-portal/` participa do fluxo
- como distinguir scripts instalados, scripts ativos e scheduler principal do host
- quais riscos existem quando a pipeline documentada diverge da pipeline real

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Portal Estático](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Dependências e origem dos dados
- [MatchZy](../02-game-panel/matchzy.md)
- [Data Sources — MatchZy SQLite](./data-sources-matchzy-sqlite.md)
- [SQL Queries and Views](./sql-queries-and-views.md)

### Saídas e publicação
- [Static API v2](./static-api-v2.md)
- [JSON Contracts](./json-contracts.md)
- [Nginx Publishing and Cache](./nginx-publishing-cache.md)
- [Frontend Structure](./portal-estatico-frontend-structure.md)

### Operação e runtime
- [Systemd Automation](../01-infra-hostinger/systemd-automation.md)
- [Nginx Static Serving](../01-infra-hostinger/nginx-static-serving.md)
- [ETL Runtime Reconciliation](./etl-runtime-reconciliation.md)
- [ETL Runtime Materialization Runbook](./etl-runtime-materialization-runbook.md)
- [ETL Repository — Minimal Shell CI](./etl-repository-minimal-shell-ci.md)
- [Operational Runbooks](./portal-estatico-operational-runbooks.md)
- [Observability and Troubleshooting](./portal-estatico-observability-troubleshooting.md)

---

## Escopo

Este documento cobre:

- o orquestrador real da pipeline v2
- scripts Bash reconciliados do host
- ordem operacional da geração principal
- locks, statefiles e base operacional do pipeline
- relação entre ETL, Nginx e árvore pública
- diferença entre automação principal e units granulares instaladas
- comandos de validação do pipeline
- problemas comuns dessa camada

Este documento não cobre em profundidade:

- schema SQL campo a campo
- contratos JSON campo a campo
- configuração completa do Nginx
- troubleshooting aprofundado do MatchZy
- operação do AMP e do servidor CS2
- Auth API no Lightsail

Esses tópicos vivem em documentos próprios dos contextos `01-infra-hostinger`, `02-game-panel`, `03-portal-estatico` e `04-infra-aws-lightsail`.

---

## Estado atual

O estado operacional conhecido e reconciliado da pipeline ETL do portal é:

- a Static API v2 é gerada por uma pipeline Bash real no host
- o orquestrador canônico atual da v2 é:
  - `/usr/local/bin/gen-all-v2.sh`
- a execução automática principal dessa pipeline acontece por:
  - `gen-all-v2.timer`
  - `gen-all-v2.service`
- a geração recorrente do `health.json` acontece separadamente por:
  - `gen-health.timer`
  - `gen-health.service`
- a base operacional do pipeline vive em:
  - `/opt/cs2-portal/`
- a árvore pública da v2 continua sendo:
  - `/var/www/api/cs2/v2/`
- a fonte versionada dos scripts ETL agora vive em `bin/` no repositório `hsc-cs2-etl`
- o runtime live do host continua sendo materializado em `/usr/local/bin/`
- o contrato do host permanece `systemd` -> `/usr/local/bin/*` -> ETL parametrizado -> `/var/www/api/cs2/v2/`
- o baseline operacional reconciliado do repositório ETL foi marcado pela tag `etl-v0.3.0`
- o repositório `hsc-cs2-etl` agora possui gate mínimo de CI para sintaxe shell e lint básico

Também ficou reconciliado no runtime real que:

- existem scripts `gen-*` adicionais instalados no host
- nem todos os scripts instalados correspondem a timers ativos
- a malha principal viva hoje é agregada, não puramente granular
- existem units granulares instaladas para `ranking`, `players`, `matches` e `content/news`, mas não são o heartbeat principal reconciliado do host

Leitura canônica:

- a pipeline da v2 hoje é **aggregator-first**
- `gen-all-v2.sh` é a fonte de verdade da orquestração principal
- qualquer leitura documental que trate outro fluxo como pipeline principal deve ser considerada stale até nova validação

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de reconciliação, são:

- runtime real da Hostinger
- `systemctl list-timers --all --no-pager`
- unit files reais em `/etc/systemd/system/`
- `ExecStart=` reconciliado de `gen-all-v2.service`
- scripts reais em `/usr/local/bin/`
- base operacional real em `/opt/cs2-portal/`
- comparação de hashes staged vs live durante a materialização
- repositório `14vosx/hsc-cs2-etl` com `bin/`, `deploy/systemd/`, `_reconcile/` e `scripts/materialize-etl-runtime.sh`
- workflow `.github/workflows/ci-shell.yml` no repositório ETL
- documentação reconciliada da Infra Hostinger e do Portal Estático

Enquanto o runtime real permanecer neste formato, essas evidências prevalecem como source of truth operacional.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/03-portal-estatico/README.md`
- `docs/03-portal-estatico/portal-estatico-architecture-runtime.md`
- `docs/03-portal-estatico/static-api-v2.md`
- `docs/03-portal-estatico/data-sources-matchzy-sqlite.md`
- `docs/03-portal-estatico/sql-queries-and-views.md`
- `docs/03-portal-estatico/nginx-publishing-cache.md`
- `docs/03-portal-estatico/portal-estatico-operational-runbooks.md`
- `docs/03-portal-estatico/portal-estatico-observability-troubleshooting.md`
- `docs/03-portal-estatico/portal-estatico-references-inventory.md`
- `docs/03-portal-estatico/etl-runtime-reconciliation.md`
- `docs/03-portal-estatico/etl-runtime-materialization-runbook.md`
- `docs/03-portal-estatico/etl-repository-minimal-shell-ci.md`
- `docs/01-infra-hostinger/systemd-automation.md`

Este documento descreve a pipeline ETL Bash da v2.  
Ele não substitui os documentos de inventário, publicação pública, runbooks ou observabilidade.

---

## Gate mínimo do repositório ETL

Além da leitura host-facing da pipeline, o repositório `hsc-cs2-etl` agora possui um gate mínimo de repositório para reduzir regressões shell triviais antes do merge.

Leitura correta desse gate:

- ele não orquestra a v2 no host
- ele não substitui a validação via `systemd`
- ele protege a camada versionada do ETL contra erro óbvio de shell

Contrato atual do gate:

- workflow `.github/workflows/ci-shell.yml`
- trigger em `push` e `pull_request`
- cobertura explícita de `bin/*.sh` e `scripts/*.sh`
- `bash -n` como checagem de sintaxe
- `shellcheck -S warning` como lint básico

Regra importante:

- `legacy/` e `_reconcile/` permanecem fora da cobertura da CI mínima
- essa fronteira não deve ser expandida sem nova decisão explícita

---

## Leitura arquitetural da pipeline

A leitura correta da pipeline pública do portal é:

1. o lado jogo produz a fonte primária
2. a fonte primária é o `matchzy.db`
3. a pipeline Bash do host lê essa fonte
4. a pipeline gera artefatos JSON públicos em `/var/www/api/cs2/v2/`
5. o Nginx publica esses artefatos sob `/api/cs2/v2/`
6. o frontend consome essa camada pública

Regra canônica:

**a pipeline ETL é a ponte operacional entre o SQLite do MatchZy e a borda pública da Static API v2**

---

## Fonte primária da pipeline

A fonte primária reconciliada da pipeline é o banco:

```bash
/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db
```

Leitura canônica:

- este é o banco vivo do runtime
- a pipeline deve ler a fonte ativa, não o backup histórico
- qualquer drift de path nessa fonte impacta diretamente a geração pública

Backup identificado, não canônico como fonte ativa:

```bash
/home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/backup-hsc/MatchZy.live-dir.pre-upstream.20260317-205843/matchzy.db
```

---

## Orquestrador canônico da v2

O orquestrador real reconciliado da pipeline principal é:

```bash
/usr/local/bin/gen-all-v2.sh
```

A unit reconciliada do host que o executa é:

- `gen-all-v2.service`

Campos relevantes reconciliados:

- `Description=Run full CS2 v2 pipeline (deterministic)`
- `User=amp`
- `Group=www-data`
- `WorkingDirectory=/var/www`
- `ExecStart=/usr/local/bin/gen-all-v2.sh`

Scheduler principal reconciliado:

- `gen-all-v2.timer`

Descrição reconciliada do timer:

- `Run full CS2 v2 pipeline every 10 minutes`

Leitura canônica:

- `gen-all-v2.sh` é a pipeline principal
- `gen-all-v2.timer` é o scheduler principal da geração da v2
- o fluxo real do host não depende hoje de timers granulares para ser a malha principal da API pública

---

## Ordem reconciliada da pipeline principal

O conteúdo reconciliado do `gen-all-v2.sh` mostra a seguinte ordem de execução principal:

1. `gen-matches.sh`
2. `gen-match-details-incremental.sh`
3. `gen-ranking.sh`
4. `gen-players-incremental.sh`
5. `gen-players-from-ranking.sh`
6. `gen-maps.sh`
7. `gen-content-news-cache.sh`
8. `gen-content-news-items-cache.sh`

Leitura canônica:

- a geração de `matches` vem antes de `match` detalhado e de `ranking`
- a pipeline já materializa também o detalhe incremental de partidas em `match/{id}.json`
- o pipeline separa:
  - geração incremental de players
  - derivação de players a partir do ranking
- `maps` entra antes do mirror same-origin de News
- o agregador principal já incorpora o refresh de `content/news/index.json` e dos itens espelhados do mirror

Essa ordem deve ser tratada como a ordem operacional viva da v2 no estado atual do host.

---

## Papel de cada script principal

## `gen-all-v2.sh`

Papel:
- orquestrador principal da v2
- coordena a geração principal da árvore pública de dados
- centraliza a execução periódica do pipeline

---

## `gen-health.sh`

Papel:
- gerar `health.json`
- validar aging e presença de artefatos principais da v2
- sustentar a observabilidade pública mínima da API estática

Importante:
- `gen-health.sh` não substitui o agregador da v2
- ele complementa o heartbeat público da camada

---

## `gen-matches.sh`

Papel:
- gerar `matches.json`
- sustentar a coleção pública de partidas
- alimentar o pipeline agregador

Observação reconciliada:
- a unit `gen-matches.service` ainda carrega descrição com `(v1)`, mas isso não muda a verdade canônica atual da camada pública, que é v2-only

---

## `gen-match-details-incremental.sh`

Papel:
- gerar `match/{id}.json` incrementalmente
- sustentar o detalhe público de partidas finalizadas
- atualizar o statefile `matches_last_matchid` do runtime

---

## `gen-ranking.sh`

Papel:
- gerar `ranking.json`
- sustentar a visão pública de ranking

---

## `gen-players-incremental.sh`

Papel:
- atualizar players de modo incremental
- reduzir custo operacional ao evitar regeneração ampla desnecessária em certos cenários

---

## `gen-players-from-ranking.sh`

Papel:
- derivar/gerar players a partir do ranking já publicado
- completar a camada pública de jogador usando `ranking.json` como insumo intermediário

Achado reconciliado:
- o script procura `ranking.json` preferencialmente em v2, mas ainda contém fallback legado para v1
- isso deve ser lido como compatibilidade residual do script, não como sinal de que v1 continua viva

---

## `gen-maps.sh`

Papel:
- gerar artefatos públicos ligados a mapas
- completar a malha pública de dados da v2

---

## `gen-content-news-cache.sh`

Papel:
- gerar `content/news/index.json` como mirror same-origin de News
- manter a camada pública do portal consumindo `content/news/` sem depender diretamente de origem cross-domain

---

## `gen-content-news-items-cache.sh`

Papel:
- espelhar os itens individuais de News sob `${ETL_BASE_DIR}/bin` no contrato atual
- complementar o índice `content/news/index.json`

---

## Scripts adicionais presentes no host

O host também possui scripts instalados que fazem parte do ecossistema operacional do portal:

- `gen-player.sh`

Leitura canônica:

- `gen-player.sh` continua sendo helper operacional explícito para geração individual de player
- `gen-match-details-incremental.sh` deixou de ser apenas presença material auxiliar e já faz parte da ordem reconciliada do agregador principal
- os scripts de content/news permanecem consumidos via `${ETL_BASE_DIR}/bin` no contrato atual

---

## Fonte versionada, runtime materializado e evidência bruta

A leitura reconciliada do runtime ETL passa a distinguir três camadas:

### 1. Fonte versionada

No repositório `hsc-cs2-etl`:

- `bin/` = fonte versionada dos scripts ETL
- `deploy/systemd/` = contrato host-facing atual das units
- `_reconcile/2026-04-01/vps-runtime/raw/` = evidência bruta da baseline importada

### 2. Runtime materializado

No host Hostinger:

- `/usr/local/bin/*.sh` = runtime materializado atualmente consumido pelo `systemd`

### 3. Base operacional

No host Hostinger:

- `/opt/cs2-portal/` = base de `locks`, `state`, `sql` e scripts de `content/news`

Leitura canônica:

- `bin/` é a origem
- `/usr/local/bin` é o runtime
- `/opt/cs2-portal` continua sendo a base operacional auxiliar do pipeline

---

## Base operacional `/opt/cs2-portal/`

A pipeline não vive apenas em `/usr/local/bin`.

A base operacional reconciliada do pipeline inclui:

- `/opt/cs2-portal/bin`
- `/opt/cs2-portal/docs`
- `/opt/cs2-portal/legacy`
- `/opt/cs2-portal/locks`
- `/opt/cs2-portal/sql`
- `/opt/cs2-portal/state`

Arquivos operacionais observados:
- `.deploy_current_tag`
- `.deploy_current_tag.prev`

Leitura canônica:

- a base `/opt/cs2-portal/` é parte estrutural do pipeline
- ela guarda lockfiles, statefiles, SQLs e scripts auxiliares do mirror de conteúdo
- qualquer troubleshooting sério da v2 precisa considerar essa base operacional

---

## Locks reconciliados

Locks reconciliados no ambiente:

- `/opt/cs2-portal/locks/gen-all-v2.lock`
- `/opt/cs2-portal/locks/gen-content-news.lock`
- `/opt/cs2-portal/locks/gen-content-news-items.lock`

Papel dos locks:
- impedir concorrência destrutiva
- proteger execuções periódicas contra overlap
- reduzir risco de artefato parcial por corrida de processos

Regra canônica:

- lock existente e ativo faz parte da operação saudável
- troubleshooting de timer disparado sem efeito útil deve considerar lock contention ou lock stale

---

## Statefiles reconciliados

Statefiles observados no ambiente:

- `/opt/cs2-portal/state/matches_last_matchid`
- `/opt/cs2-portal/state/players_last_matchid`

Leitura canônica:

- a pipeline possui estado operacional persistido fora da árvore pública
- isso reforça que a geração não é puramente “stateless”
- perda ou incoerência desses statefiles pode afetar incrementalidade e reconciliação dos geradores

---

## SQLs versionados reconciliados

SQLs observados na base operacional:

- `/opt/cs2-portal/sql/indexes.sql`
- `/opt/cs2-portal/sql/player_v2.sql`
- `/opt/cs2-portal/sql/ranking.sql`
- `/opt/cs2-portal/sql/ranking_v2.sql`

Leitura canônica:

- a pipeline tem base SQL versionada materializada no host
- isso sustenta a geração pública da v2
- a camada SQL não deve ser tratada como detalhe invisível da pipeline

---

## Árvore pública de saída

A árvore pública reconciliada da v2 é:

```bash
/var/www/api/cs2/v2/
```

Leitura canônica:

- é aqui que os artefatos públicos finais precisam existir
- o Nginx publica essa árvore sob:
  - `/api/cs2/v2/`
- a pipeline ETL não se completa apenas quando os scripts rodam; ela se completa quando os artefatos públicos finais ficam corretos nessa árvore

---

## Relação entre pipeline agregada e units granulares

O host possui units granulares instaladas para:

- `gen-matches`
- `gen-players`
- `gen-ranking`
- `gen-content-news`
- `gen-content-news-items`

Mas a leitura canônica atual é:

- essas units **existem**
- algumas têm timers instalados, porém desativados
- elas **não** são o scheduler principal vivo da v2 hoje
- o scheduler principal vivo é a dupla:
  - `gen-all-v2.timer`
  - `gen-health.timer`

Essa distinção é crítica, porque evita dois erros comuns:

1. tratar toda unit instalada como parte ativa do heartbeat principal  
2. confundir trilha histórica ou fallback com a automação viva do runtime

---

## Mirror de `content/news`

A base operacional também possui scripts específicos para o mirror same-origin de News:

- `/opt/cs2-portal/bin/gen-content-news-cache.sh`
- `/opt/cs2-portal/bin/gen-content-news-items-cache.sh`

Services reconciliados:

- `gen-content-news.service`
- `gen-content-news-items.service`

Timers correspondentes:
- existem no host
- estão desativados no estado atual reconciliado

Leitura canônica:

- o mirror de News tem automação materializada
- mas essa automação não faz parte do heartbeat principal ativo da v2
- isso precisa ser lido como capacidade instalada, não como pipeline central obrigatória de cada ciclo de 10 minutos

---

## Heartbeat operacional da camada

O heartbeat real da camada pública, hoje, é:

### A cada 10 minutos
- `gen-all-v2.timer`
- `gen-all-v2.service`
- `/usr/local/bin/gen-all-v2.sh`

### A cada 1 minuto
- `gen-health.timer`
- `gen-health.service`
- `/usr/local/bin/gen-health.sh`

Leitura canônica:

- essa é a malha principal de automação viva reconciliada no host
- qualquer runbook do portal deve partir desse desenho

---

## Comandos de validação

Os comandos abaixo representam a validação mínima da pipeline ETL.

### Validar timers principais

```bash
systemctl list-timers --all --no-pager | grep -Ei 'gen|v2|health|portal|hsc'
```

### Validar units relevantes

```bash
systemctl list-unit-files --type=service --no-pager | grep -Ei 'gen|v2|health|portal|hsc'
systemctl list-unit-files --type=timer --no-pager | grep -Ei 'gen|v2|health|portal|hsc'
```

### Validar metadados principais das units

```bash
grep -RInE 'Description=|ExecStart=|WorkingDirectory=|User=|Group=|OnCalendar=|WantedBy=' /etc/systemd /lib/systemd/system 2>/dev/null | grep -Ei 'gen|v2|health|portal|hsc'
```

### Validar scripts principais no host

```bash
ls -lah /usr/local/bin | grep -Ei 'gen|v2|health|portal|hsc'
```

### Validar base operacional do portal

```bash
ls -lah /opt/cs2-portal
find /opt/cs2-portal -maxdepth 2 -type f | sort
```

### Validar referências internas do agregador

```bash
grep -RInE 'gen-all|gen-health|gen-ranking|gen-matches|gen-players|gen-maps|health.json|ranking.json|matches.json' /usr/local/bin /opt/cs2-portal 2>/dev/null
```

### Validar fonte primária da pipeline

```bash
ls -l /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db
sqlite3 /home/amp/.ampdata/instances/MixHAXIXE01/counter-strike2/730/game/csgo/addons/counterstrikesharp/plugins/MatchZy/matchzy.db ".tables"
```

### Validar artefatos públicos principais

```bash
curl -I https://haxixesmokeclub.com/api/cs2/v2/health.json
curl -I https://haxixesmokeclub.com/api/cs2/v2/ranking.json
curl -I https://haxixesmokeclub.com/api/cs2/v2/matches.json
```

---

## Problemas comuns

### 1. Timer ativo, mas a v2 continua stale

Causas comuns:
- lock impedindo execução útil
- script quebrado
- fonte SQLite stale
- etapa intermediária falhando dentro do agregador

Impacto:
- a automação parece viva
- o resultado público continua degradado

---

### 2. Artefatos públicos existem, mas `health.json` acusa problema

Causas comuns:
- arquivos antigos demais
- aging acima do limite esperado
- falha recente do agregador
- `gen-health.sh` refletindo corretamente um problema real da v2

Impacto:
- a observabilidade pública acusa degradação antes que um operador perceba visualmente o problema

---

### 3. Operador dispara unit granular achando que recompõe a v2 inteira

Causas comuns:
- leitura antiga da automação
- confusão entre capacidade instalada e heartbeat principal
- documentação stale

Impacto:
- regeneração parcial
- falsa sensação de correção
- v2 continua inconsistente em outras superfícies

---

### 4. `gen-players-from-ranking.sh` falha por dependência no ranking

Causas comuns:
- `ranking.json` não gerado
- ranking stale
- fluxo intermediário quebrado

Impacto:
- camada de player degrada
- parte da v2 fica inconsistente mesmo que outras superfícies ainda respondam

---

### 5. Fonte SQLite saudável, mas árvore pública não reflete isso

Causas comuns:
- pipeline não executou
- script falhou
- problema de permissão/publicação
- drift entre working directory, output path e borda

Impacto:
- problema parece “do portal”
- mas a raiz está na ponte ETL/publicação

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- a fonte principal da pipeline é o `matchzy.db`
- o orquestrador canônico atual da v2 é `/usr/local/bin/gen-all-v2.sh`
- o scheduler principal da v2 é `gen-all-v2.timer`
- o scheduler principal do `health.json` é `gen-health.timer`
- a base operacional do pipeline vive em `/opt/cs2-portal/`
- a árvore pública da v2 vive em `/var/www/api/cs2/v2/`
- units granulares podem existir no host sem serem o heartbeat principal atual
- a pipeline viva precisa ser lida como geração agregada + health separado

Esses invariantes ajudam a preservar a leitura correta do ETL público do portal.

---

## Limites deste documento

Este documento não detalha:

- conteúdo completo linha a linha de todos os scripts Bash
- lógica SQL completa de cada geração
- schema JSON detalhado
- troubleshooting profundo do Nginx
- troubleshooting profundo do MatchZy
- política futura de reativação de timers granulares

Esses tópicos pertencem a documentos complementares ou a futuras reconciliações finas.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- o orquestrador real da v2 estiver explícito sem ambiguidade
- a ordem reconciliada da pipeline estiver clara
- locks, statefiles e base operacional estiverem corretamente posicionados
- a distinção entre heartbeat principal e units granulares instaladas estiver formalizada
- ele puder ser usado como referência confiável do ETL Bash do Portal Estático sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: portal estático / ETL Bash pipeline
- Última revisão: 2026-03-18