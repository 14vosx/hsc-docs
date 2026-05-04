# systemd Automation

## Objetivo

Documentar a camada de automação `systemd` da Infra Hostinger do ecossistema HSC, registrando os timers e services reais do host que sustentam a geração da Static API v2, do `health.json` e das rotinas auxiliares já materializadas no ambiente.

Este documento existe para registrar, de forma estável e auditável:

- quais timers e services realmente existem no host
- quais automações estão ativas hoje
- qual é o comando agregador real da pipeline v2
- quais unidades permanecem instaladas, mas não ativas ou não fazem parte da malha principal
- como locks, working directories e usuários do host entram nessa automação
- quais riscos existem quando a automação documentada diverge da automação real

---

## Navegação

### Entrada
- [Home da documentação](../README.md)
- [Infra Hostinger](./README.md)
- [Master Index](../00-governance/99-master-index.md)

### Infraestrutura imediata
- [Architecture Runtime](./infra-hostinger-architecture-runtime.md)
- [Docker Host](./docker-host.md)
- [Filesystem Paths and Permissions](./filesystem-paths-permissions.md)
- [Nginx Static Serving](./nginx-static-serving.md)

### Geração e publicação de artefatos
- [ETL Bash Pipeline](../03-portal-estatico/etl-bash-pipeline.md)
- [Static API v2](../03-portal-estatico/static-api-v2.md)
- [Nginx Publishing and Cache](../03-portal-estatico/nginx-publishing-cache.md)
- [Operational Runbooks](../03-portal-estatico/portal-estatico-operational-runbooks.md)
- [ETL Runtime Reconciliation](../03-portal-estatico/etl-runtime-reconciliation.md)
- [ETL Runtime Materialization Runbook](../03-portal-estatico/etl-runtime-materialization-runbook.md)

### Operação e suporte
- [Observability and Troubleshooting](./infra-hostinger-observability-troubleshooting.md)
- [Network, DNS and TLS](././infra-hostinger-network-dns-tls.md)
- [Documentation System](../00-governance/documentation-system.md)

---

## Escopo

Este documento cobre:

- timers e services `systemd` relevantes do lado Hostinger
- geração recorrente da Static API v2
- geração recorrente do `health.json`
- unidades instaladas para geração granular e mirror de conteúdo
- working directories, locks e comandos reais reconciliados
- validações operacionais da camada de automação
- riscos e problemas comuns dessa automação

Este documento não cobre em profundidade:

- implementação linha a linha de cada script Bash
- contratos JSON da v2
- configuração completa do Nginx
- operação do AMP e do servidor CS2
- Auth API canônica no Lightsail
- troubleshooting aprofundado de conteúdo/news

Esses tópicos vivem em documentos próprios dos contextos `01-infra-hostinger`, `02-game-panel`, `03-portal-estatico` e `04-infra-aws-lightsail`.

---

## Estado atual

O estado operacional conhecido e reconciliado da automação `systemd` do lado Hostinger é:

- a automação principal da v2 está centrada em:
  - `gen-all-v2.timer`
  - `gen-all-v2.service`
- a automação dedicada de News está ativa em modo conservador:
  - `gen-content-news.timer`
  - `gen-content-news-items.timer`
  - `gen-content-news.service`
  - `gen-content-news-items.service`
- a geração recorrente do `health.json` está centrada em:
  - `gen-health.timer`
  - `gen-health.service`
- existem units granulares instaladas para:
  - ranking
  - players
  - matches
  - content/news
  - content/news items
- as units granulares de News foram ativadas de forma conservadora para reduzir latência pública sem priorizar News acima da estabilidade do jogo
- o comando agregador real da v2 já foi reconciliado como:
  - `/usr/local/bin/gen-all-v2.sh`

Também ficou reconciliado no runtime real que:

- `gen-health.timer` está ativo
- `gen-all-v2.timer` está ativo
- `gen-content-news.timer` está `enabled/active`
- `gen-content-news-items.timer` está `enabled/active`
- os timers granulares de `matches`, `players` e `ranking` existem, mas não estão ativos no estado atual
- o host possui scripts reais em `/usr/local/bin/`
- o host possui base operacional do portal em `/opt/cs2-portal/`

Contexto operacional conservador:

- a VPS Hostinger também roda o servidor CS2 via Game Panel/AMP
- a estabilidade do jogo tem prioridade sobre a latência de News
- o plano observado no momento da validação era 2 vCPU, cerca de 8 GiB RAM, sem swap e disco em torno de 70%
- há upgrade previsto em cerca de 20 dias para Game Panel 4, com 4 vCPU, 16 GiB RAM e 200 GB NVMe
- a decisão atual é não reduzir os timers de News para 60s antes desse upgrade
- a cadência deve ser reavaliada depois do Game Panel 4

Leitura canônica:

- a v2 hoje é gerada por pipeline agregada
- a saúde pública da v2 depende principalmente de `gen-all-v2` + `gen-health`
- News possui timers dedicados ativos em modo conservador
- os demais timers granulares sobrevivem como infraestrutura instalada, mas não como scheduler principal ativo

---

## Source of truth / evidências

As principais evidências deste documento, nesta fase de reconciliação, são:

- runtime real da Hostinger
- saída de `systemctl list-timers --all --no-pager`
- saída de `systemctl list-unit-files`
- unit files reais em `/etc/systemd/system/`
- campos reais de `Description=`, `ExecStart=`, `WorkingDirectory=`, `User=`, `Group=` e `WantedBy=`
- scripts reais em `/usr/local/bin/`
- estrutura operacional real em `/opt/cs2-portal/`

Enquanto o runtime real permanecer neste formato, essas evidências prevalecem como source of truth operacional.

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/infra-hostinger-architecture-runtime.md`
- `docs/01-infra-hostinger/infra-hostinger-references-inventory.md`
- `docs/01-infra-hostinger/infra-hostinger-observability-troubleshooting.md`
- `docs/03-portal-estatico/etl-bash-pipeline.md`
- `docs/03-portal-estatico/portal-estatico-operational-runbooks.md`
- `ddocs/03-portal-estatico/portal-estatico-references-inventory.md`

Este documento descreve a automação do host.  
Ele não substitui os documentos do pipeline ETL, dos runbooks da v2 ou do inventário do portal.

---

## Units reais reconciliadas

As units reconciliadas no host para a camada portal/v2 são:

### Timers ativos

- `gen-health.timer`
- `gen-all-v2.timer`
- `gen-content-news.timer`
- `gen-content-news-items.timer`

### Services principais ativos por ativação indireta

- `gen-health.service`
- `gen-all-v2.service`
- `gen-content-news.service`
- `gen-content-news-items.service`

### Timers instalados, mas não ativos no estado atual

- `gen-matches.timer`
- `gen-players.timer`
- `gen-ranking.timer`

### Services granulares instalados

- `gen-content-news.service`
- `gen-content-news-items.service`
- `gen-matches.service`
- `gen-players.service`
- `gen-ranking.service`

Leitura canônica:

- a presença dessas units no host está confirmada
- a ativação principal atual da v2 depende dos timers agregados
- as units granulares de News agora são scheduler dedicado vivo do mirror de News
- as demais units granulares não devem ser tratadas como scheduler principal vivo sem evidência de ativação futura

---

## Timers ativos do runtime

A fotografia reconciliada do host mostrou estes timers relevantes ativos:

### `gen-health.timer`

Descrição reconciliada:
- `Run health generator every 1 minute`

Ativa:
- `gen-health.service`

Papel:
- gerar `health.json` com cadência curta
- manter a observabilidade pública da v2 atualizada

---

### `gen-all-v2.timer`

Descrição reconciliada:
- `Run full CS2 v2 pipeline every 10 minutes`

Ativa:
- `gen-all-v2.service`

Papel:
- disparar a pipeline agregada da Static API v2
- regenerar os artefatos públicos principais da camada de dados

Leitura canônica:

- o heartbeat operacional do portal hoje é:
  - `gen-health` a cada 1 minuto
  - `gen-all-v2` a cada 10 minutos
  - `gen-content-news` em modo conservador para lista de News
  - `gen-content-news-items` em modo conservador para detalhes de News

---

## Services principais reconciliados

### `gen-health.service`

Descrição reconciliada:
- `Generate CS2 API health.json`

Campos reconciliados:
- `User=amp`
- `Group=www-data`
- `ExecStart=/usr/local/bin/gen-health.sh`

Papel:
- regenerar `health.json`
- validar idade e presença de artefatos-chave
- alimentar a superfície pública de health da v2

Leitura canônica:
- esta unit pertence à camada pública do portal/v2
- não deve ser lida como health da Auth API no Lightsail

---

### `gen-all-v2.service`

Descrição reconciliada:
- `Run full CS2 v2 pipeline (deterministic)`

Campos reconciliados:
- `User=amp`
- `Group=www-data`
- `WorkingDirectory=/var/www`
- `ExecStart=/usr/local/bin/gen-all-v2.sh`
- `WantedBy=multi-user.target`

Papel:
- executar a pipeline principal agregada da v2
- coordenar geração de matches, ranking, players e maps
- centralizar a regeneração pública do lado portal

Leitura canônica:
- esta é a unit principal do pipeline da v2
- o comando agregador do host já não é hipótese; ele está reconciliado diretamente no runtime

---

## Comando agregador real da v2

O comando agregador real reconciliado do host é:

```bash
/usr/local/bin/gen-all-v2.sh
```

O script reconciliado utiliza:

- `TAG="gen-all-v2"`
- lock:
  - `/opt/cs2-portal/locks/gen-all-v2.lock`

E executa, em ordem reconciliada, os passos:

1. `gen-matches.sh`
2. `gen-ranking.sh`
3. `gen-players-incremental.sh`
4. `gen-players-from-ranking.sh`
5. `gen-maps.sh`

Leitura canônica:

- este é o pipeline agregador real do runtime atual
- qualquer documentação que trate outro comando como orchestrator principal da v2 deve ser considerada stale até nova validação

---

## Units granulares instaladas

O host ainda possui units granulares instaladas para partes específicas da geração.

### `gen-matches.service`

Descrição reconciliada:
- `Generate MatchZy matches.json (v1)`

Campos reconciliados:
- `User=amp`
- `Group=www-data`
- `ExecStart=/usr/local/bin/gen-matches.sh`

Observação importante:
- a descrição ainda carrega a expressão `(v1)`
- isso deve ser lido como resquício histórico da própria unit, não como prova de que a v1 continua canônica
- a camada pública oficial segue sendo a v2

---

### `gen-players.service`

Descrição reconciliada:
- `Generate MatchZy player JSONs (v2)`

Campos reconciliados:
- `User=amp`
- `Group=www-data`
- `ExecStart=/usr/local/bin/gen-players-incremental.sh`

---

### `gen-ranking.service`

Descrição reconciliada:
- `Generate MatchZy ranking.json (v2)`

Campos reconciliados:
- `User=amp`
- `Group=www-data`
- `ExecStart=/bin/bash -lc '/usr/local/bin/gen-ranking.sh && /usr/local/bin/gen-players-from-ranking.sh'`

Leitura canônica:
- esta unit já mostra que ranking e player derivado já foram acoplados em uma mesma execução no host

---

## Units de content/news

O host também possui automação instalada para o mirror same-origin de News.

### `gen-content-news.service`

Descrição reconciliada:
- `HSC: Generate content/news cache (mirror from Auth API)`

Campos reconciliados:
- `User=amp`
- `Group=www-data`
- `WorkingDirectory=/opt/cs2-portal`
- `ExecStart=/usr/bin/flock -n /opt/cs2-portal/locks/gen-content-news.lock /opt/cs2-portal/bin/gen-content-news-cache.sh`

---

### `gen-content-news-items.service`

Descrição reconciliada:
- `HSC: Generate content/news item caches (mirror by slug)`

Campos reconciliados:
- `User=amp`
- `Group=www-data`
- `WorkingDirectory=/opt/cs2-portal`
- `ExecStart=/usr/bin/flock -n /opt/cs2-portal/locks/gen-content-news-items.lock /opt/cs2-portal/bin/gen-content-news-items-cache.sh`

Leitura canônica:
- o host já possui automação materializada para o mirror de News
- seus timers dedicados agora estão ativos em modo conservador
- antes dessa ativação, News dependia do `gen-all-v2.timer` para atualização automática, com latência maior

### `gen-content-news.timer`

Estado reconciliado:
- `enabled/active`

Configuração conservadora atual:
- `OnUnitActiveSec=120s`
- drop-in com `OnActiveSec=120s`

Drop-in reconciliado:
- `/etc/systemd/system/gen-content-news.timer.d/override.conf`

Papel:
- atualizar o index público de News
- reduzir a latência do mirror para até ~2 min para lista

### `gen-content-news-items.timer`

Estado reconciliado:
- `enabled/active`

Configuração conservadora atual:
- `OnUnitActiveSec=300s`
- drop-in com `OnActiveSec=300s`

Drop-in reconciliado:
- `/etc/systemd/system/gen-content-news-items.timer.d/override.conf`

Papel:
- atualizar os detalhes públicos de News por `slug`
- reduzir a latência do mirror para até ~5 min para detalhes

### Motivo dos drop-ins

Após `enable`, os timers dedicados de News ficaram `active elapsed` com `Trigger n/a`.

O uso de `OnActiveSec` nos drop-ins passou a garantir a primeira execução e o reagendamento após restart do timer.

### Validação observada

Validação de execução:

- `gen-content-news.service` medido com elapsed de aproximadamente `0.603s`
- `gen-content-news-items.service` medido com elapsed de aproximadamente `1.020s`
- execução automática dos dois timers validada
- logs com `ok`
- load após execução observado em `0.02, 0.01, 0.00`

Leitura correta:

- o impacto observado no momento do teste foi baixo
- isso não deve ser lido como garantia de ausência de impacto no servidor CS2
- a cadência atual permanece conservadora até o upgrade previsto da VPS

---

## Timers granulares instalados, mas não ativos

A fotografia do host confirmou timers existentes, porém não ativos no estado atual:

- `gen-matches.timer`
- `gen-players.timer`
- `gen-ranking.timer`

Leitura canônica:

- essas units fazem parte da infraestrutura instalada
- não devem ser tratadas como heartbeat principal atual
- podem servir como trilha histórica, fallback ou automação reservada, mas isso exigiria reconciliação operacional adicional

Observação:

- `gen-content-news.timer` e `gen-content-news-items.timer` deixaram este grupo após a ativação conservadora validada

---

## Scripts reconciliados no host

O host possui os seguintes scripts principais reconciliados em `/usr/local/bin/`:

- `gen-all-v2.sh`
- `gen-health.sh`
- `gen-maps.sh`
- `gen-match-details-incremental.sh`
- `gen-matches.sh`
- `gen-players-from-ranking.sh`
- `gen-player.sh`
- `gen-players-incremental.sh`
- `gen-ranking.sh`

Leitura canônica:

- a família `gen-*` está materializada no host
- nem todos os scripts presentes estão necessariamente ligados a timers ativos
- o inventário de scripts em disco é maior do que a malha principal ativa reconciliada

---

## Base operacional do portal

A base operacional reconciliada do pipeline no host inclui:

### Diretório base

- `/opt/cs2-portal/`

### Subárvores observadas

- `/opt/cs2-portal/bin`
- `/opt/cs2-portal/docs`
- `/opt/cs2-portal/legacy`
- `/opt/cs2-portal/locks`
- `/opt/cs2-portal/sql`
- `/opt/cs2-portal/state`

### Arquivos operacionais observados

- `.deploy_current_tag`
- `.deploy_current_tag.prev`

### Locks reconciliados

- `/opt/cs2-portal/locks/gen-all-v2.lock`
- `/opt/cs2-portal/locks/gen-content-news.lock`
- `/opt/cs2-portal/locks/gen-content-news-items.lock`

### Statefiles reconciliados

- `/opt/cs2-portal/state/matches_last_matchid`
- `/opt/cs2-portal/state/players_last_matchid`

Leitura canônica:

- o pipeline da v2 não vive só em `/usr/local/bin`
- ele depende explicitamente da base operacional de `/opt/cs2-portal/`

---

## SQLs versionados reconciliados

A fotografia do host confirmou a presença de SQLs versionados em:

- `/opt/cs2-portal/sql/indexes.sql`
- `/opt/cs2-portal/sql/player_v2.sql`
- `/opt/cs2-portal/sql/ranking.sql`
- `/opt/cs2-portal/sql/ranking_v2.sql`

Leitura canônica:

- a camada SQL versionada existe materialmente no host
- isso reforça que o ETL público possui base operacional organizada fora do Nginx e fora do runtime do jogo

---

## Relação entre automação do host e portal público

A relação operacional correta é:

- `gen-all-v2.timer` aciona `gen-all-v2.service`
- `gen-all-v2.service` executa `/usr/local/bin/gen-all-v2.sh`
- `gen-all-v2.sh` gera os artefatos públicos principais da v2
- `gen-health.timer` aciona `gen-health.service`
- `gen-health.service` executa `/usr/local/bin/gen-health.sh`
- `gen-content-news.timer` aciona `gen-content-news.service`
- `gen-content-news.service` executa `/opt/cs2-portal/bin/gen-content-news-cache.sh`
- `gen-content-news-items.timer` aciona `gen-content-news-items.service`
- `gen-content-news-items.service` executa `/opt/cs2-portal/bin/gen-content-news-items-cache.sh`
- o Nginx publica os artefatos em `/var/www/api/cs2/v2/`

Leitura canônica:

- o pipeline de publicação pública do portal depende de `systemd`
- a saúde da v2 depende diretamente dessa automação
- problema em timer/service/script pode parecer problema de Nginx ou de JSON público

---

## Achado importante de drift residual no host

A fotografia do host também confirmou a presença de:

- `hsc-auth-api.service`

Campos reconciliados:
- `Description=HSC Auth API (Node/Express)`
- `User=hscapi`
- `WorkingDirectory=/opt/hsc-auth-api`
- `ExecStart=/usr/bin/node /opt/hsc-auth-api/index.js`

Leitura correta:

- a Auth API canônica está hoje no Lightsail
- a presença desta unit na Hostinger deve ser tratada como drift residual / legado operacional
- ela não deve ser promovida como runtime canônico atual da Auth API

Este ponto deve ser tratado como item de cleanup técnico e de inventário da Hostinger, não como refutação da arquitetura canônica atual.

---

## Comandos de validação

Os comandos abaixo representam a validação mínima desta camada.

### Validar timers relevantes do host

```bash
systemctl list-timers --all --no-pager | grep -Ei 'gen|v2|health|hsc|portal'
```

### Validar unit files relevantes

```bash
systemctl list-unit-files --type=service --no-pager | grep -Ei 'gen|v2|health|hsc|portal'
systemctl list-unit-files --type=timer --no-pager | grep -Ei 'gen|v2|health|hsc|portal'
```

### Validar timers dedicados de News

```bash
systemctl status gen-content-news.timer gen-content-news-items.timer --no-pager
systemctl list-timers --all --no-pager | grep -Ei 'gen-content-news'
```

### Validar drop-ins dos timers de News

```bash
systemctl cat gen-content-news.timer
systemctl cat gen-content-news-items.timer
```

### Validar services dedicados de News

```bash
systemctl status gen-content-news.service gen-content-news-items.service --no-pager
journalctl -u gen-content-news.service -u gen-content-news-items.service --no-pager -n 80
```

### Validar localização dos unit files

```bash
find /etc/systemd /lib/systemd/system -maxdepth 2 -type f | grep -Ei 'gen|v2|health|hsc|portal' | sort
```

### Validar campos principais das units

```bash
grep -RInE 'Description=|ExecStart=|WorkingDirectory=|User=|Group=|OnCalendar=|WantedBy=' /etc/systemd /lib/systemd/system 2>/dev/null | grep -Ei 'gen|v2|health|hsc|portal'
```

### Validar scripts principais do pipeline

```bash
ls -lah /usr/local/bin | grep -Ei 'gen|v2|health|portal|hsc'
```

### Validar base operacional do portal

```bash
ls -lah /opt/cs2-portal
find /opt/cs2-portal -maxdepth 2 -type f | sort
```

### Validar referências internas ao agregador real

```bash
grep -RInE 'gen-all|gen-health|gen-ranking|gen-matches|gen-players|gen-maps|health.json|ranking.json|matches.json' /usr/local/bin /opt/cs2-portal 2>/dev/null
```

---

## Rollback operacional dos timers dedicados de News

Use este rollback se os timers dedicados de News apresentarem erro recorrente, contenção operacional ou impacto observado indesejado no host.

### Rollback conservador

Desabilita apenas o agendamento dedicado de News e preserva os services para execução manual controlada:

```bash
sudo systemctl disable --now gen-content-news.timer gen-content-news-items.timer
sudo systemctl reset-failed gen-content-news.timer gen-content-news-items.timer gen-content-news.service gen-content-news-items.service
```

Validação após rollback:

```bash
systemctl list-timers --all --no-pager | grep -Ei 'gen-content-news'
systemctl is-enabled gen-content-news.timer gen-content-news-items.timer
systemctl is-active gen-content-news.timer gen-content-news-items.timer
```

Leitura esperada:

- timers dedicados de News deixam de agendar automaticamente
- `gen-content-news.service` e `gen-content-news-items.service` continuam disponíveis para execução manual
- News volta a depender da automação agregada e/ou de execução manual controlada, com latência maior

### Rollback completo dos drop-ins

Remove também os drop-ins de cadência conservadora:

```bash
sudo systemctl disable --now gen-content-news.timer gen-content-news-items.timer
sudo rm -f /etc/systemd/system/gen-content-news.timer.d/override.conf
sudo rm -f /etc/systemd/system/gen-content-news-items.timer.d/override.conf
sudo systemctl daemon-reload
sudo systemctl reset-failed gen-content-news.timer gen-content-news-items.timer gen-content-news.service gen-content-news-items.service
```

Validação após rollback completo:

```bash
systemctl cat gen-content-news.timer
systemctl cat gen-content-news-items.timer
systemctl list-timers --all --no-pager | grep -Ei 'gen-content-news'
```

Regra operacional:

- não reduzir para 60s antes do upgrade de VPS
- se houver dúvida entre latência de News e estabilidade do jogo, preservar a estabilidade do Game Panel/AMP
- reavaliar cadência e necessidade dos drop-ins depois do Game Panel 4

---

## Problemas comuns

### 1. Timer ativo, script quebrado

Causas comuns:

- alteração manual de script
- dependência quebrada
- lock não liberado
- path esperado ausente

Impacto:
- a unit existe e agenda corretamente
- mas a v2 não atualiza como esperado

---

### 2. Unit granular existe, mas não faz parte do fluxo vivo

Causas comuns:

- legado operacional
- transição para pipeline agregada
- automação mantida como fallback ou trilha histórica

Impacto:
- documentação pode superestimar relevância de uma unit não ativa
- troubleshooting fica confuso se o operador assume que toda unit instalada está em uso real

---

### 3. Drift entre service description e arquitetura atual

Causas comuns:

- descrição antiga não revisada
- permanência de naming herdado
- evolução do pipeline sem renome completo de units

Exemplo reconciliado:
- `gen-matches.service` ainda cita `(v1)`

Impacto:
- o texto da unit pode induzir leitura errada
- por isso o runtime precisa ser interpretado junto com o contexto canônico

---

### 4. Lock impede execução concorrente

Causas comuns:

- execução sobreposta
- processo anterior preso
- lockfile stale

Impacto:
- timer dispara
- mas execução útil não acontece

---

### 5. Drift residual de Auth API na Hostinger

Causas comuns:

- runtime antigo preservado
- cleanup incompleto após migração para Lightsail
- service legado ainda habilitado no host

Impacto:
- confusão arquitetural
- manutenção no host errado
- leitura documental incorreta se esse drift não for explicitado

---

## Invariantes operacionais

Os invariantes conhecidos desta camada incluem:

- o heartbeat principal do portal é hoje:
  - `gen-health.timer`
  - `gen-all-v2.timer`
- o mirror de News possui timers dedicados ativos em modo conservador:
  - `gen-content-news.timer`
  - `gen-content-news-items.timer`
- o comando agregador real da v2 é:
  - `/usr/local/bin/gen-all-v2.sh`
- o lock principal do agregador é:
  - `/opt/cs2-portal/locks/gen-all-v2.lock`
- a base operacional do pipeline vive em:
  - `/opt/cs2-portal/`
- os scripts `gen-*` existem materialmente em `/usr/local/bin/`
- as units granulares de News estão ativas para reduzir latência pública do mirror de News
- as demais units granulares existem, mas não são o scheduler principal ativo reconciliado
- o host ainda possui drift residual de `hsc-auth-api.service`, que não deve ser tratado como runtime canônico atual
- a estabilidade do Game Panel/AMP tem prioridade sobre reduzir latência de News

Esses invariantes ajudam a preservar a leitura correta da automação do host.

---

## Limites deste documento

Este documento não detalha:

- o conteúdo completo de cada unit file
- o conteúdo linha a linha dos scripts Bash
- troubleshooting profundo de cada generator
- política completa de retry/alerta da automação
- cleanup operacional da Auth API residual na Hostinger
- política futura para reativação dos timers granulares que não são News
- decisão pós-upgrade sobre reduzir ou não a cadência de News

Esses tópicos pertencem a documentos complementares ou a futuras reconciliações finas.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os timers e services ativos do host estiverem explícitos sem ambiguidade
- o comando agregador real da v2 estiver formalizado
- locks, working directories e usuários estiverem reconciliados com o runtime
- as units granulares instaladas, porém não ativas, estiverem corretamente posicionadas
- o drift residual da Auth API na Hostinger estiver explicitado
- ele puder ser usado como referência confiável da automação `systemd` do lado Hostinger sem depender do master legado

---
---

## Contrato atual entre `systemd` e runtime materializado

A leitura canônica consolidada nesta frente passa a ser:

- `bin/` no repositório ETL é a fonte versionada
- `/usr/local/bin/*.sh` é a camada materializada de runtime no host
- `systemd` continua apontando para `/usr/local/bin/*`
- `${ETL_BASE_DIR}` continua sustentando `locks`, `state`, `sql` e scripts de `content/news`

Motivo desta decisão:

- as units atuais já rodam como `amp:www-data`
- o host já possui hardening, `ReadWritePaths` e paths operacionais consolidados
- retargetar `ExecStart` diretamente para a árvore do repositório nesta fase aumentaria risco sem ganho proporcional

Leitura canônica:

- o `systemd` continua host-facing
- a materialização passa a ser a ponte entre o Git e o runtime real

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infra-hostinger / systemd
- Última revisão: 2026-05-04
