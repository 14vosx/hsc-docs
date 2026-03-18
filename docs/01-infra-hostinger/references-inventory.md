# References and Inventory

## Objetivo

Consolidar o inventário de referências, evidências, artefatos operacionais, paths críticos e itens pendentes de validação do contexto Infra Hostinger do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- quais documentos alimentam este contexto
- quais artefatos reais do host são source of truth operacional
- quais paths e componentes são críticos para a camada-base
- quais comandos ajudam a validar rapidamente a infraestrutura
- quais pontos ainda dependem de confirmação direta no ambiente real
- quais são os limites documentais deste contexto

---

## Escopo

Este documento cobre:

- documentos de origem do contexto
- artefatos reais do host Hostinger
- serviços principais da camada-base
- paths críticos conhecidos
- componentes estruturais relevantes
- comandos de validação
- itens pendentes de confirmação
- limites documentais do contexto

Este documento não cobre em profundidade:

- explicação detalhada da operação do AMP
- runbooks do servidor CS2
- troubleshooting aprofundado do ETL
- contratos JSON da Static API v2
- operação da Auth API no Lightsail
- histórico completo de mudanças

Esses assuntos vivem nos documentos especializados dos contextos `02-game-panel`, `03-portal-estatico` e `04-infra-aws-lightsail`, bem como em impl-logs e material legado.

---

## Estado atual

O contexto `01-infra-hostinger` já possui estrutura canônica definida para documentar:

- arquitetura de runtime da VPS Hostinger
- borda pública via Nginx
- DNS, TLS e Certbot do lado Hostinger
- Docker como substrate do host
- systemd e automações do host
- filesystem, paths e permissões
- observabilidade e troubleshooting da camada-base

Neste estágio da migração, o contexto já foi estruturado documentalmente, mas ainda depende de reconciliação progressiva com o host real para fechar alguns pontos específicos de hostname público final, nomes exatos de units/timers e detalhes finos da camada de borda.

---

## Source of truth / evidências

As evidências que sustentam este contexto se dividem em quatro grupos principais.

### 1. Documentação consolidada do ecossistema

Usada para:

- visão macro do papel da Hostinger no HSC
- separação entre Infra Hostinger, Game Panel, Portal Estático e Lightsail
- topologia geral do lado game + portal

### 2. Documentação específica da camada Hostinger

Usada para:

- runtime do host Debian
- publishing público do portal e da v2
- uso de Nginx, Certbot, Docker e systemd
- paths e áreas operacionais do host

### 3. Impl-logs e registros incrementais

Usados para:

- rastrear mudanças em publishing, paths, cache, assets e automações
- registrar ajustes operacionais do lado Hostinger
- preservar rastreabilidade de correções relevantes de infraestrutura

### 4. Ambiente real do host

Usado para:

- validar paths efetivos
- validar domínio/hostname público vigente
- confirmar serviços e timers reais
- confirmar ownership e grupos
- confirmar a topologia real de Docker, Nginx e automação

Regra principal:

- quando houver conflito entre documento histórico e ambiente real validado, prevalece o ambiente real validado

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/01-infra-hostinger/README.md`
- `docs/01-infra-hostinger/architecture-runtime.md`
- `docs/01-infra-hostinger/network-dns-tls.md`
- `docs/01-infra-hostinger/nginx-static-serving.md`
- `docs/01-infra-hostinger/docker-host.md`
- `docs/01-infra-hostinger/certbot.md`
- `docs/01-infra-hostinger/systemd-automation.md`
- `docs/01-infra-hostinger/filesystem-paths-permissions.md`
- `docs/01-infra-hostinger/observability-troubleshooting.md`

Este documento não substitui nenhum dos arquivos acima.  
Ele funciona como fechamento de inventário e referência do contexto.

---

## Documentos de origem

Os documentos abaixo são as principais fontes de extração e reconciliação deste contexto.

### Fontes primárias

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação específica da infraestrutura Hostinger
- documentação específica da camada AMP/CS2 que ajuda a revelar dependências estruturais do host

### Fontes secundárias

- documentação reconciliada do Portal Estático
- documentação reconciliada do Game Panel
- impl-logs ligados a publishing, mirror same-origin, paths, permissões e automação

### Fontes de apoio histórico

- master antigo
- blueprint histórico
- documentação legada do lado Hostinger
- materiais `_old` úteis para reconciliação de paths, topologia e decisões operacionais

Regra documental:

- fontes históricas ajudam a reconciliar
- mas não governam o canônico sozinhas

---

## Artefatos reais do host

Os artefatos reais conhecidos ou esperados do host Hostinger para este contexto incluem:

### Host e sistema

- VPS Hostinger
- Debian

### Edge público

- Nginx
- borda pública do portal
- borda pública da Static API v2

### TLS

- Certbot
- certificados do lado Hostinger
- exposição HTTPS do domínio público do portal

### Substrate de containers

- Docker
- daemon Docker no host

### Automação

- `systemd`
- timers e services do host ligados à geração/publicação do lado portal

### Filesystem público

- `/var/www/portal/cs2/`
- `/var/www/api/cs2/v2/`

### Filesystem operacional

- `/opt/cs2-portal/`
- `/opt/cs2-portal/locks/`
- `/opt/cs2-portal/state/`
- `/opt/cs2-portal/sql/`
- `/usr/local/bin/`

### Base estrutural do lado AMP

- `/home/amp/.ampdata/instances/`

Esses artefatos devem ser tratados como inventário-base do contexto até revisão explícita.

---

## Serviços principais do contexto

Os serviços principais conhecidos desta camada são:

### `nginx`

Papel:
- edge público do lado Hostinger
- serving do portal e da Static API v2

### `docker`

Papel:
- substrate do lado game
- sustentação do runtime containerizado associado ao Game Panel

### `systemd`

Papel:
- orquestração de services e timers do host
- suporte à automação recorrente da camada pública

### Certbot e renovação de certificados

Papel:
- manutenção da camada HTTPS do lado Hostinger

Esses componentes formam o núcleo mínimo da infraestrutura base do lado game + portal.

---

## Paths críticos

Os paths críticos conhecidos deste contexto incluem:

### Publicação do portal

- `/var/www/portal/cs2/`

### Publicação da Static API v2

- `/var/www/api/cs2/v2/`

### Base operacional do portal

- `/opt/cs2-portal/`

### Locks

- `/opt/cs2-portal/locks/`

### Lock global principal

- `/opt/cs2-portal/locks/gen-all-v2.lock`

### Statefiles

- `/opt/cs2-portal/state/`

### SQLs versionados

- `/opt/cs2-portal/sql/`

### Scripts operacionais

- `/usr/local/bin/`

### Base estrutural da instância AMP

- `/home/amp/.ampdata/instances/`

Regra prática:

- qualquer mudança nesses paths deve ser tratada como alteração relevante e refletida no canônico e, quando necessário, no impl-log

---

## Componentes estruturais relevantes

Os componentes estruturais mais importantes já reconciliados neste contexto são:

### Debian

Base do host que sustenta serviços, paths, automação e publishing.

### Nginx

Borda pública do lado Hostinger.

### Certbot

Suporte de TLS da borda pública.

### Docker host

Substrate do lado game.

### systemd + timers

Automação recorrente do host.

### Filesystem público

Camada efetivamente servida ao usuário final.

### Filesystem operacional

Camada onde vivem ETL, locks, statefiles, SQLs e scripts.

### Árvore AMP

Base estrutural do lado game, relevante para dependências indiretas do portal.

---

## Ownership, grupos e leitura pública

As convenções de ownership e grupo já reconciliadas como relevantes incluem a coexistência operacional entre:

- `amp`
- `www-data`

Esse ponto é importante porque:

- `amp` preserva o lado operacional do runtime do jogo e da árvore AMP
- `www-data` representa a leitura pública do Nginx
- a camada pública do host depende de artefatos que precisam ser gerados por uma parte do sistema e lidos por outra

Regra importante:

- qualquer drift de owner/group em paths críticos pode quebrar publicação, ETL ou serving mesmo com serviços aparentemente saudáveis

---

## Workloads dependentes desta base

Os principais workloads sustentados por esta infraestrutura são:

### Lado Game Panel

- AMP
- instância `MixHAXIXE01`
- servidor CS2
- origem do `matchzy.db`

### Lado Portal Estático

- portal público
- Static API v2
- ETL Bash
- publishing público
- same-origin de conteúdo, quando aplicável

Regra arquitetural:

- este documento inventaria a base
- a semântica operacional dos workloads vive em seus próprios contextos

---

## Comandos de validação

Os comandos abaixo formam um kit mínimo de validação da camada-base Hostinger.

### Validar Nginx

```bash
sudo nginx -t
sudo systemctl status nginx
sudo journalctl -u nginx -n 100 --no-pager
```

### Validar Docker

```bash
sudo systemctl status docker
docker ps --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}"
sudo journalctl -u docker -n 100 --no-pager
```

### Validar timers e services do host

```bash
systemctl list-timers --all
```

### Validar árvores públicas

```bash
ls -lah /var/www/portal/cs2/
ls -lah /var/www/api/cs2/v2/
```

### Validar artefato público da v2

```bash
jq . /var/www/api/cs2/v2/health.json
```

### Validar árvores operacionais

```bash
ls -lah /opt/cs2-portal/
ls -lah /opt/cs2-portal/locks/
ls -lah /opt/cs2-portal/state/
ls -lah /opt/cs2-portal/sql/
ls -lah /usr/local/bin/
```

### Validar árvore AMP

```bash
ls -lah /home/amp/.ampdata/instances/
```

### Validar acesso público ao portal

Substitua pelo domínio público vigente.

```bash
curl -I https://SEU_DOMINIO/
```

### Validar acesso público à v2

Substitua pelo path público real da v2.

```bash
curl -I https://SEU_DOMINIO/SEU_PATH_PUBLICO_DA_V2/health.json
```

### Validar camada TLS

```bash
certbot --version
sudo certbot certificates
```

---

## Itens pendentes de confirmação

Os itens abaixo ainda devem ser confirmados diretamente no ambiente real para elevar o grau de confiança do contexto.

### 1. Hostname público canônico final do lado Hostinger

É necessário fixar com confirmação operacional qual é o domínio/hostname produtivo oficial vigente do portal e da v2.

### 2. Nomes exatos dos timers e services do lado portal

A intenção operacional da automação está reconciliada, mas os nomes finais de todas as units relevantes ainda precisam ser confirmados diretamente no host.

### 3. Root/alias final exato dos vhosts públicos

Os paths em filesystem estão reconciliados.  
A confirmação final da forma como o Nginx aponta para eles ainda deve ser fechada no ambiente real.

### 4. Convenção final de ownership e grupo dos diretórios mais sensíveis

A lógica entre `amp` e `www-data` está clara, mas a fotografia final exata dos diretórios mais críticos ainda pode ser refinada com validação direta do host.

### 5. Detalhes finos da renovação automática de certificados

A presença do Certbot e o papel da camada TLS estão reconciliados, mas o mecanismo exato já configurado no host ainda pode ser explicitado melhor após validação direta.

### 6. Eventuais logs adicionais fora de `journalctl`

É útil confirmar, no host real, se existem logs locais específicos da camada Hostinger que devam ser referenciados formalmente neste contexto.

---

## Itens fora do escopo deste contexto

Os itens abaixo não pertencem ao inventário canônico da Infra Hostinger:

- runbooks de scrim, veto, coach e warmup
- operação detalhada do AMP
- MatchZy em profundidade
- pipeline ETL em detalhe operacional fino
- contratos JSON campo a campo
- frontend do portal em detalhe
- Auth API no Lightsail
- credenciais reais
- arquivos de acesso sensíveis
- documentação histórica não reconciliada

Esses itens pertencem a outros contextos ou devem permanecer fora do fluxo normal do repositório documental.

---

## Limites documentais do contexto

Os limites documentais deste contexto são:

- ele documenta a camada-base da VPS Hostinger
- ele não documenta sozinho os workloads superiores
- ele não substitui o índice mestre
- ele não substitui impl-logs históricos
- ele não deve absorver material sensível
- ele depende de confirmação periódica contra o host real para permanecer confiável

---

## Critério de manutenção

Este documento deve ser atualizado quando houver:

- mudança de path crítico
- mudança de hostname público oficial do lado Hostinger
- mudança de serviço estrutural da camada
- mudança relevante de ownership/grupo em áreas críticas
- alteração relevante do inventário de automação do host
- confirmação ou resolução de item pendente listado aqui

Mudanças pequenas de comportamento de um workload superior devem ser refletidas primeiro no documento especializado correspondente, e não necessariamente aqui.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os artefatos reais do host estiverem confirmados sem ambiguidade
- os paths críticos estiverem validados no ambiente real
- o hostname público oficial do lado Hostinger estiver fixado
- os itens pendentes estiverem resolvidos ou explicitamente mantidos como pendência consciente
- ele puder ser usado como inventário confiável da camada-base sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura Hostinger / references and inventory
- Última revisão: 2026-03-18