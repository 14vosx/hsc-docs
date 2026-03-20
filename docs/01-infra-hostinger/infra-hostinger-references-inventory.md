# References and Inventory

## Objetivo

Consolidar o inventário de referências, evidências, artefatos operacionais, paths críticos e itens pendentes de validação do contexto Infra Hostinger do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- quais documentos alimentam este contexto
- quais artefatos reais do host são source of truth operacional
- quais componentes, serviços e paths são críticos para a camada Hostinger
- quais comandos ajudam a validar rapidamente a infraestrutura base
- quais pontos ainda dependem de confirmação direta no ambiente real
- quais são os limites documentais deste contexto

---

## Escopo

Este documento cobre:

- documentos de origem do contexto
- artefatos reais do host Hostinger
- serviços principais da camada base
- hostnames e arquivos de Nginx conhecidos
- timers, services e automações relevantes
- paths críticos de filesystem
- comandos de validação
- itens pendentes de confirmação
- limites documentais do contexto

Este documento não cobre em profundidade:

- operação completa do servidor CS2
- inventário detalhado de plugins do Game Panel
- ETL detalhado da Static API v2
- Auth API canônica no Lightsail
- credenciais e arquivos sensíveis
- histórico completo de mudanças

Esses assuntos vivem nos documentos especializados de outros contextos, bem como em impl-logs e material legado.

---

## Estado atual

O contexto `01-infra-hostinger` já possui estrutura canônica definida para documentar:

- substrate Debian do host
- Nginx como edge do portal e da v2
- Docker host do lado jogo
- Certbot / TLS
- automação via `systemd`
- filesystem, paths e permissões
- observabilidade e troubleshooting da camada base

Neste estágio da reconciliação, o contexto já possui confirmação operacional suficiente para fixar sem ambiguidade:

- hostname técnico do host:
  - `srv1353392.hstgr.cloud`
- hostnames públicos canônicos do lado Hostinger:
  - `haxixesmokeclub.com`
  - `www.haxixesmokeclub.com`
- hostnames não ativos para esta borda no estado atual:
  - `portal.haxixesmokeclub.com`
  - `api.haxixesmokeclub.com`
- arquivo principal de configuração Nginx reconciliado do host:
  - `/etc/nginx/conf.d/srv1353392.hstgr.cloud.conf`
- timers ativos principais da camada portal/v2:
  - `gen-health.timer`
  - `gen-all-v2.timer`
- service agregador real da v2:
  - `gen-all-v2.service`
- comando agregador real da v2:
  - `/usr/local/bin/gen-all-v2.sh`

Também ficou reconciliado que o host ainda contém drift residual ligado à Auth API, incluindo:

- configuração Nginx com `server_name auth-api.haxixesmokeclub.com`
- unit `hsc-auth-api.service`

Leitura canônica:

- a Hostinger continua sendo substrate de portal + v2 + lado jogo
- a Auth API canônica não pertence mais a esta infraestrutura
- qualquer artefato de Auth API neste host deve ser tratado como residual/stale até cleanup explícito

---

## Source of truth / evidências

As evidências que sustentam este contexto se dividem em quatro grupos principais.

### 1. Documentação consolidada do ecossistema

Usada para:

- visão macro do papel da Hostinger no HSC
- separação entre Hostinger, Game Panel, Portal Estático e Lightsail
- topologia geral do lado game + portal

### 2. Documentação específica da camada Hostinger

Usada para:

- Nginx
- Docker host
- Certbot
- systemd
- filesystem e permissões
- observabilidade da infraestrutura base

### 3. Impl-logs e registros incrementais

Usados para:

- rastrear mudanças em publishing, automação, paths, assets e endurecimento operacional
- registrar ajustes relevantes do substrate da Hostinger
- preservar rastreabilidade da evolução do host

### 4. Ambiente real da Hostinger

Usado para:

- validar hostname técnico e hostnames públicos reais
- validar configuração Nginx
- validar paths públicos e internos
- validar timers/services reais
- validar scripts em `/usr/local/bin`
- validar a base operacional em `/opt/cs2-portal/`
- confirmar o estado real do substrate do host

Regra principal:

- quando houver conflito entre documento histórico e ambiente real validado, prevalece o ambiente real validado

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/infra-hostinger-architecture-runtime.md`
- `docs/01-infra-hostinger/./infra-hostinger-network-dns-tls.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/docker-host.md`
- `docs/01-infra-hostinger/certbot.md`
- `docs/01-infra-hostinger/systemd-automation.md`
- `docs/01-infra-hostinger/filesystem-paths-permissions.md`
- `docs/01-infra-hostinger/infra-hostinger-observability-troubleshooting.md`
- `ddocs/03-portal-estatico/portal-estatico-references-inventory.md`
- `docs/02-game-panel/game-panel-references-inventory.md`

Este documento não substitui nenhum dos arquivos acima.  
Ele funciona como fechamento de inventário e referência do contexto.

---

## Documentos de origem

Os documentos abaixo são as principais fontes de extração e reconciliação deste contexto.

### Fontes primárias

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação específica da Infra Hostinger
- documentação reconciliada da camada Nginx/systemd/publicação

### Fontes secundárias

- documentação reconciliada do Portal Estático
- documentação reconciliada do Game Panel
- impl-logs ligados a automação, publishing, assets e endurecimento da borda

### Fontes de apoio histórico

- master antigo
- blueprint histórico
- documentação legada da VPS Hostinger
- materiais `_old` úteis para reconciliar naming, paths, timers e vhosts

Regra documental:

- fontes históricas ajudam a reconciliar
- mas não governam o canônico sozinhas

---

## Artefatos reais do host

Os artefatos reais conhecidos ou esperados do host Hostinger incluem:

### Host e sistema

- VPS Hostinger
- Debian
- hostname técnico:
  - `srv1353392.hstgr.cloud`

### Hostnames públicos canônicos

- `haxixesmokeclub.com`
- `www.haxixesmokeclub.com`

### Hostnames não ativos para esta borda

- `portal.haxixesmokeclub.com`
- `api.haxixesmokeclub.com`

### Borda pública

- Nginx
- arquivo principal reconciliado:
  - `/etc/nginx/conf.d/srv1353392.hstgr.cloud.conf`

### Inventário de Nginx observado

- `/etc/nginx/nginx.conf`
- `/etc/nginx/conf.d/srv1353392.hstgr.cloud.conf`
- `/etc/nginx/sites-available/default`
- `/etc/nginx/sites-available/redirect_to_https`
- `/etc/nginx/sites-available/auth-api.haxixesmokeclub.com`

### Automação `systemd`

- `gen-all-v2.service`
- `gen-all-v2.timer`
- `gen-health.service`
- `gen-health.timer`
- units granulares instaladas para `matches`, `players`, `ranking` e `content/news`

### Scripts reconciliados no host

- `/usr/local/bin/gen-all-v2.sh`
- `/usr/local/bin/gen-health.sh`
- `/usr/local/bin/gen-maps.sh`
- `/usr/local/bin/gen-match-details-incremental.sh`
- `/usr/local/bin/gen-matches.sh`
- `/usr/local/bin/gen-players-from-ranking.sh`
- `/usr/local/bin/gen-player.sh`
- `/usr/local/bin/gen-players-incremental.sh`
- `/usr/local/bin/gen-ranking.sh`

### Base operacional do portal

- `/opt/cs2-portal/`
- `/opt/cs2-portal/bin/`
- `/opt/cs2-portal/locks/`
- `/opt/cs2-portal/sql/`
- `/opt/cs2-portal/state/`

### Locks reconciliados

- `/opt/cs2-portal/locks/gen-all-v2.lock`
- `/opt/cs2-portal/locks/gen-content-news.lock`
- `/opt/cs2-portal/locks/gen-content-news-items.lock`

### Statefiles reconciliados

- `/opt/cs2-portal/state/matches_last_matchid`
- `/opt/cs2-portal/state/players_last_matchid`

### Raízes públicas relevantes

- `/var/www/portal/cs2/`
- `/var/www/api/cs2/v2/`

### Árvore AMP / jogo

- `/home/amp/.ampdata/instances/`

Esses artefatos devem ser tratados como inventário-base do contexto até revisão explícita.

---

## Serviços principais do contexto

Os serviços principais conhecidos desta camada são:

### `nginx`

Papel:
- edge HTTP/HTTPS do lado Hostinger
- publicação do portal e da v2
- terminação TLS do lado portal/game

### `docker`

Papel:
- substrate do workload do lado jogo
- apoio operacional ao container `AMP_MixHAXIXE01`

### `certbot`

Papel:
- renovação de certificados do lado Hostinger

### `gen-all-v2.service`

Papel:
- pipeline agregada principal da Static API v2

### `gen-health.service`

Papel:
- geração recorrente do `health.json`

### units granulares de geração

Papel:
- infraestrutura instalada para geração segmentada
- suporte técnico e histórico da pipeline

### `hsc-auth-api.service` (residual)

Papel atual correto:
- drift residual / legado operacional
- não runtime canônico da Auth API

---

## Paths críticos

Os paths críticos conhecidos deste contexto incluem:

### Configuração principal do Nginx

- `/etc/nginx/conf.d/srv1353392.hstgr.cloud.conf`

### Árvore principal do Nginx

- `/etc/nginx/`

### Portal público

- `/var/www/portal/cs2/`

### Static API v2 pública

- `/var/www/api/cs2/v2/`

### Base operacional do portal

- `/opt/cs2-portal/`

### Scripts operacionais

- `/usr/local/bin/`

### Locks do pipeline

- `/opt/cs2-portal/locks/`

### Statefiles do pipeline

- `/opt/cs2-portal/state/`

### SQLs versionados

- `/opt/cs2-portal/sql/`

### Árvore das instâncias AMP

- `/home/amp/.ampdata/instances/`

### Unit residual da Auth API

- `/etc/systemd/system/hsc-auth-api.service`

Regra prática:

- qualquer mudança nesses paths ou na relação entre borda, automação e publicação deve ser tratada como alteração relevante e refletida no canônico e, quando necessário, no impl-log

---

## Componentes estruturais relevantes

Os componentes estruturais mais importantes já reconciliados neste contexto são:

### VPS Hostinger

Base física/lógica do lado game + portal do ecossistema.

### Debian

Sistema operacional do substrate.

### Nginx

Borda pública e camada de publicação do lado Hostinger.

### Docker host

Substrate do lado jogo.

### Certbot

Sustentação da camada TLS.

### systemd

Heartbeat operacional da automação do host.

### Base operacional `/opt/cs2-portal/`

Estrutura viva da pipeline pública do portal/v2.

### Família `gen-*`

Scripts reais do pipeline e geração pública.

---

## Componentes já reconciliados

Os itens abaixo já possuem relevância reconciliada suficiente no contexto atual.

### Hostname técnico do host

- `srv1353392.hstgr.cloud`

### Hostnames públicos canônicos do lado Hostinger

- `haxixesmokeclub.com`
- `www.haxixesmokeclub.com`

### Edge pública principal do host

- Nginx
- `/etc/nginx/conf.d/srv1353392.hstgr.cloud.conf`

### Timers ativos principais do lado portal/v2

- `gen-health.timer`
- `gen-all-v2.timer`

### Services principais da automação pública

- `gen-health.service`
- `gen-all-v2.service`

### Comando agregador real da v2

- `/usr/local/bin/gen-all-v2.sh`

### Base operacional reconciliada

- `/opt/cs2-portal/`

### Locks reconciliados

- `gen-all-v2.lock`
- `gen-content-news.lock`
- `gen-content-news-items.lock`

### Drift residual identificado

- vhost/config residual de `auth-api.haxixesmokeclub.com` na Hostinger
- `hsc-auth-api.service` ainda presente no host

Esses itens já devem ser tratados como parte da verdade operacional do contexto.

---

## Dependências cruzadas

Os principais workloads e dependências cruzadas deste contexto incluem:

### Dependência do lado portal

A publicação pública depende de:

- Nginx íntegro
- paths públicos corretos
- timers/services funcionando
- scripts `gen-*` íntegros
- base `/opt/cs2-portal/` saudável

### Dependência do lado jogo

O substrate da Hostinger depende de:

- Docker íntegro
- árvore AMP íntegra
- persistência do lado CS2 saudável

### Dependência de TLS

O host depende de:

- Nginx
- Certbot
- hostnames públicos reais corretos

### Dependência de separação arquitetural

A leitura correta do ecossistema depende de:

- Hostinger = jogo + portal + v2
- Lightsail = Auth API
- qualquer resquício de Auth API na Hostinger = drift residual

---

## Comandos de validação

Os comandos abaixo formam um kit mínimo de validação da camada Hostinger.

### Validar hostname técnico

```bash
hostname -f
```

### Validar hostnames públicos principais

```bash
curl -I https://haxixesmokeclub.com/
curl -I https://www.haxixesmokeclub.com/
```

### Validar sintaxe do Nginx

```bash
sudo nginx -t
```

### Validar configuração relevante do Nginx

```bash
sudo nginx -T | grep -nE "server_name|root |alias "
sudo nginx -T | sed -n '200,320p'
```

### Validar inventário de arquivos do Nginx

```bash
find /etc/nginx -maxdepth 3 -type f | sort
```

### Validar timers relevantes do host

```bash
systemctl list-timers --all --no-pager | grep -Ei 'gen|v2|health|hsc|portal'
```

### Validar unit files relevantes

```bash
systemctl list-unit-files --type=service --no-pager | grep -Ei 'gen|v2|health|hsc|portal'
systemctl list-unit-files --type=timer --no-pager | grep -Ei 'gen|v2|health|hsc|portal'
```

### Validar campos principais das units

```bash
grep -RInE 'Description=|ExecStart=|WorkingDirectory=|User=|Group=|OnCalendar=|WantedBy=' /etc/systemd /lib/systemd/system 2>/dev/null | grep -Ei 'gen|v2|health|hsc|portal'
```

### Validar scripts principais do host

```bash
ls -lah /usr/local/bin | grep -Ei 'gen|v2|health|portal|hsc'
```

### Validar base operacional do portal

```bash
ls -lah /opt/cs2-portal
find /opt/cs2-portal -maxdepth 2 -type f | sort
```

### Validar workload Docker do lado jogo

```bash
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
```

---

## Itens pendentes de confirmação

Os itens abaixo ainda devem ser confirmados diretamente no ambiente real para elevar o grau de confiança do contexto.

### 1. Root/alias final completo de todos os blocos públicos relevantes

Os blocos principais já estão reconciliados, mas ainda há ganho possível em congelar a fotografia final completa do vhost público.

### 2. Política fina de renovação/certificados em runtime

A camada Certbot já está corretamente posicionada, mas detalhes finos do fluxo de renovação ainda podem ser refinados no documento específico.

### 3. Papel atual exato das units granulares desativadas

Elas já estão reconciliadas como instaladas, mas o papel operacional pretendido delas ainda pode ser explicitado melhor no futuro.

### 4. Cleanup técnico do drift residual da Auth API na Hostinger

O drift já foi identificado.  
Ainda falta decidir e executar a limpeza técnica correspondente no host.

### 5. Auditoria de eventuais backups/configs antigas do Nginx

Já há evidência de arquivos `.bak` na árvore de configuração do host.  
Uma auditoria futura ainda pode classificá-los melhor.

---

## Itens fora do escopo deste contexto

Os itens abaixo não pertencem ao inventário canônico da Infra Hostinger:

- runtime canônico da Auth API no Lightsail
- MariaDB local da Auth API
- inventário detalhado dos plugins carregados no Game Panel
- contratos JSON da v2
- frontend em nível de componente
- credenciais reais
- arquivos de acesso sensíveis
- documentação histórica não reconciliada

Esses itens pertencem a outros contextos ou devem permanecer fora do fluxo normal do repositório documental.

---

## Limites documentais do contexto

Os limites documentais deste contexto são:

- ele documenta o substrate Hostinger
- ele não documenta sozinho o lado jogo em detalhe
- ele não documenta sozinho o Portal Estático em detalhe
- ele não substitui o índice mestre
- ele não substitui impl-logs históricos
- ele depende de confirmação periódica contra o ambiente real para permanecer confiável

---

## Critério de manutenção

Este documento deve ser atualizado quando houver:

- mudança de hostname público canônico do lado Hostinger
- mudança relevante na estrutura Nginx do host
- mudança do comando agregador da v2
- mudança relevante na base `/opt/cs2-portal/`
- mudança relevante na malha de timers/services do host
- confirmação ou resolução de item pendente listado aqui
- cleanup do drift residual da Auth API neste host

Mudanças pequenas de comportamento funcional devem ser refletidas primeiro no documento especializado correspondente, e não necessariamente aqui.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os artefatos reais do host estiverem confirmados sem ambiguidade
- hostnames, configuração principal do Nginx e automação `systemd` estiverem claramente reconciliados
- a base operacional do portal estiver corretamente posicionada
- o drift residual da Auth API na Hostinger estiver explicitado
- os itens pendentes estiverem resolvidos ou explicitamente mantidos como pendência consciente
- ele puder ser usado como inventário confiável da Infra Hostinger sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura Hostinger / references and inventory
- Última revisão: 2026-03-18