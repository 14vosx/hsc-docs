# References and Inventory

## Objetivo

Consolidar o inventário de referências, evidências, artefatos operacionais, paths críticos e itens pendentes de validação do contexto Infra AWS Lightsail do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- quais documentos alimentam este contexto
- quais artefatos reais do host são source of truth operacional
- quais serviços, paths e componentes são críticos para a Auth API
- quais comandos ajudam a validar rapidamente a infraestrutura do Lightsail
- quais pontos ainda dependem de confirmação direta no ambiente real
- quais são os limites documentais deste contexto

---

## Escopo

Este documento cobre:

- documentos de origem do contexto
- artefatos reais do host Lightsail
- serviços principais da camada
- paths críticos conhecidos
- componentes estruturais relevantes
- comandos de validação
- itens pendentes de confirmação
- limites documentais do contexto

Este documento não cobre em profundidade:

- infraestrutura base da Hostinger
- pipeline ETL detalhado da Static API v2
- contratos JSON do portal
- operação do servidor CS2
- credenciais e arquivos sensíveis
- histórico completo de mudanças

Esses assuntos vivem nos documentos especializados dos outros contextos, bem como em impl-logs e material legado.

---

## Estado atual

O contexto `04-infra-aws-lightsail` já possui estrutura canônica definida para documentar:

- arquitetura de runtime da instância Lightsail
- borda pública da Auth API via Nginx reverse proxy
- DNS, TLS e hostname canônico da Auth API
- Node.js via systemd
- MariaDB local
- deploy, release e rollback
- backup e restore
- observabilidade e troubleshooting da camada dinâmica

Neste estágio da reconciliação, o contexto já possui confirmação operacional suficiente para fixar sem ambiguidade:

- a Auth API canônica no Lightsail
- o hostname público canônico `auth-api.haxixesmokeclub.com`
- o reverse proxy Nginx no próprio host Lightsail
- o upstream local do Nginx em `127.0.0.1:3000`
- a unit canônica da aplicação:
  - `hsc-auth-api.service`
- o arquivo real da unit:
  - `/etc/systemd/system/hsc-auth-api.service`
- o usuário real de execução:
  - `hscadmin`
- o working directory real:
  - `/opt/hsc/hsc-auth-api`
- o `ExecStart` real:
  - `/usr/bin/node index.js`
- a política de restart:
  - `Restart=always`
- o ambiente explicitamente reconciliado:
  - `NODE_ENV=production`
- o binding observado da aplicação:
  - `0.0.0.0:3000`
- a presença do vhost reconciliado em:
  - `/etc/nginx/sites-available/hsc-auth-api`
- a resposta pública bem-sucedida de `/health`
- o script real de backup do MariaDB:
  - `/opt/hsc/backup-mariadb.sh`
- o diretório real dos dumps:
  - `/opt/hsc/backups/mariadb/`
- o log real da camada de backup:
  - `/opt/hsc/backups/mariadb/backup.log`

Ainda restam pendências menores, principalmente ligadas a:

- mecanismo exato de agendamento do backup
- política exata de retenção dos dumps
- procedimento de restore validado de ponta a ponta
- eventual confirmação de outras variáveis de ambiente fora da unit
- cleanup do drift residual da borda antiga ainda presente na Hostinger

---

## Source of truth / evidências

As evidências que sustentam este contexto se dividem em quatro grupos principais.

### 1. Documentação consolidada do ecossistema

Usada para:

- visão macro do papel do Lightsail no HSC
- separação entre Hostinger, Game Panel, Portal Estático e Auth API
- topologia geral da camada dinâmica

### 2. Documentação específica da Auth API

Usada para:

- runtime da aplicação
- Nginx reverse proxy
- MariaDB local
- deploy/release/rollback
- backup e restore
- observabilidade da camada dinâmica

### 3. Impl-logs e registros incrementais

Usados para:

- rastrear mudanças em deploy, borda, backup e operação da Auth API
- registrar ajustes relevantes de infraestrutura e aplicação
- preservar rastreabilidade da evolução da camada dinâmica

### 4. Ambiente real do Lightsail

Usado para:

- validar hostname canônico
- validar reverse proxy real
- validar upstream local da aplicação
- validar unit `systemd` real
- validar usuário, working directory e `ExecStart`
- validar processo Node e porta local
- validar arquivos de Nginx no host
- validar o endpoint `/health`
- validar script, diretório e log reais de backup
- confirmar o estado real do runtime da Auth API

Regra principal:

- quando houver conflito entre documento histórico e ambiente real validado, prevalece o ambiente real validado

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/architecture-runtime.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/network-dns-tls.md`
- `docs/04-infra-aws-lightsail/nginx-reverse-proxy.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`
- `docs/04-infra-aws-lightsail/backup-restore.md`

Este documento não substitui nenhum dos arquivos acima.  
Ele funciona como fechamento de inventário e referência do contexto.

---

## Documentos de origem

Os documentos abaixo são as principais fontes de extração e reconciliação deste contexto.

### Fontes primárias

- documentação consolidada atual do ecossistema HSC
- blueprint técnico consolidado do HSC
- documentação específica da infraestrutura AWS Lightsail
- documentação específica da Auth API

### Fontes secundárias

- documentação reconciliada de borda, Node.js, MariaDB local e backup
- impl-logs ligados a deploy, rollback, borda pública e ajustes operacionais da API

### Fontes de apoio histórico

- master antigo
- blueprint histórico
- documentação legada da Auth API
- materiais `_old` úteis para reconciliar hostnames, paths, rotinas de deploy e mudanças de runtime

Regra documental:

- fontes históricas ajudam a reconciliar
- mas não governam o canônico sozinhas

---

## Artefatos reais do host

Os artefatos reais conhecidos ou esperados do host Lightsail para este contexto incluem:

### Host e sistema

- instância AWS Lightsail
- Ubuntu 22.04 LTS

### Borda pública

- Nginx
- reverse proxy da Auth API

### Hostname público canônico

- `auth-api.haxixesmokeclub.com`

### Upstream do Nginx

- `http://127.0.0.1:3000`

### Binding observado da aplicação

- `0.0.0.0:3000`

### Vhost reconciliado da API

- `/etc/nginx/sites-available/hsc-auth-api`

### Configuração default do host

- `/etc/nginx/sites-available/default`

### Unit canônica da aplicação

- `hsc-auth-api.service`

### Arquivo real da unit

- `/etc/systemd/system/hsc-auth-api.service`

### Processo da aplicação

- serviço Node.js via systemd
- aplicação executada com:
  - `/usr/bin/node index.js`

### Working directory real

- `/opt/hsc/hsc-auth-api`

### Usuário real da unit

- `hscadmin`

### Persistência local

- MariaDB local do contexto da Auth API

### Script real de backup

- `/opt/hsc/backup-mariadb.sh`

### Diretório real dos dumps

- `/opt/hsc/backups/mariadb/`

### Log real da camada de backup

- `/opt/hsc/backups/mariadb/backup.log`

### Dumps reais observados

- `hsc_auth_*.sql.gz`

Esses artefatos devem ser tratados como inventário-base do contexto até revisão explícita.

---

## Serviços principais do contexto

Os serviços principais conhecidos desta camada são:

### `nginx`

Papel:
- borda pública da Auth API
- terminação TLS
- reverse proxy para o runtime local

### `hsc-auth-api.service`

Papel:
- unit canônica da aplicação
- sustentação do runtime Node.js da Auth API
- integração da app ao boot e à operação normal do host

### processo Node.js da Auth API

Papel:
- aplicação dinâmica do ecossistema
- runtime local atrás do reverse proxy

### `mariadb.service`

Papel:
- persistência relacional da Auth API
- storage principal da camada dinâmica
- dependência direta da camada de backup

### `systemd`

Papel:
- sustentação do serviço da aplicação
- apoio operacional para start, stop, restart e observabilidade

### script `backup-mariadb.sh`

Papel:
- entrypoint real reconciliado da camada de backup do banco
- geração dos dumps `.sql.gz`
- alimentação do diretório real de backups da Auth API

Esses componentes formam o núcleo mínimo da infraestrutura dinâmica do Lightsail.

---

## Paths críticos

Os paths críticos conhecidos deste contexto incluem:

### Vhost reconciliado da Auth API

- `/etc/nginx/sites-available/hsc-auth-api`

### Vhost default do host

- `/etc/nginx/sites-available/default`

### Árvore de configuração do Nginx

- `/etc/nginx/`

### Unit canônica da aplicação

- `/etc/systemd/system/hsc-auth-api.service`

### Working directory real da app

- `/opt/hsc/hsc-auth-api`

### Upstream do Nginx

- `127.0.0.1:3000`

### Binding observado da app

- `0.0.0.0:3000`

### Script real de backup

- `/opt/hsc/backup-mariadb.sh`

### Diretório real dos dumps

- `/opt/hsc/backups/mariadb/`

### Log real da camada de backup

- `/opt/hsc/backups/mariadb/backup.log`

Regra prática:

- qualquer mudança nesses paths ou nessa relação entre hostname, proxy, unit, runtime Node e backup do banco deve ser tratada como alteração relevante e refletida no canônico e, quando necessário, no impl-log

---

## Componentes estruturais relevantes

Os componentes estruturais mais importantes já reconciliados neste contexto são:

### AWS Lightsail

Base do host da camada dinâmica do ecossistema.

### Ubuntu 22.04 LTS

Sistema operacional do runtime atual da Auth API.

### Nginx reverse proxy

Borda pública da API.

### Hostname canônico

- `auth-api.haxixesmokeclub.com`

### Unit canônica da aplicação

- `hsc-auth-api.service`

### Working directory real

- `/opt/hsc/hsc-auth-api`

### Execução real da aplicação

- `/usr/bin/node index.js`

### Usuário real

- `hscadmin`

### Upstream do Nginx

- `127.0.0.1:3000`

### Binding observado da aplicação

- `0.0.0.0:3000`

### MariaDB local

Persistência principal do backend dinâmico.

### Camada real de backup

- `/opt/hsc/backup-mariadb.sh`
- `/opt/hsc/backups/mariadb/`
- `/opt/hsc/backups/mariadb/backup.log`

### Endpoint `/health`

Ponto mínimo de validação pública e operacional da borda e da app.

---

## Componentes já reconciliados

Os itens abaixo já possuem relevância reconciliada suficiente no contexto atual.

### Hostname canônico da API

- `auth-api.haxixesmokeclub.com`

### Reverse proxy local

- Nginx no próprio Lightsail

### Upstream do Nginx

- `http://127.0.0.1:3000`

### Unit canônica da app

- `hsc-auth-api.service`

### Arquivo real da unit

- `/etc/systemd/system/hsc-auth-api.service`

### Working directory real

- `/opt/hsc/hsc-auth-api`

### Usuário real

- `hscadmin`

### ExecStart real

- `/usr/bin/node index.js`

### Binding observado da app

- `0.0.0.0:3000`

### Vhost reconciliado

- `/etc/nginx/sites-available/hsc-auth-api`

### Script real de backup

- `/opt/hsc/backup-mariadb.sh`

### Diretório real dos dumps

- `/opt/hsc/backups/mariadb/`

### Log real da camada de backup

- `/opt/hsc/backups/mariadb/backup.log`

### Endpoint público validado

- `https://auth-api.haxixesmokeclub.com/health`

Esses itens já devem ser tratados como parte da verdade operacional do contexto.

---

## Dependências cruzadas

Os principais workloads e dependências cruzadas deste contexto incluem:

### Dependência do DNS e da borda

A Auth API depende de:

- resolução correta de `auth-api.haxixesmokeclub.com`
- Nginx íntegro
- TLS saudável
- proxy coerente com o upstream real

### Dependência da unit e da aplicação local

A borda depende de:

- `hsc-auth-api.service` íntegra
- working directory correto
- processo Node funcional
- porta `3000` respondendo localmente
- serviço local estável via systemd

### Dependência do banco local

A aplicação depende de:

- MariaDB local íntegro
- credenciais e configuração operacional corretas
- compatibilidade de schema com a versão atual da app

### Dependência da camada de backup

A resiliência do contexto depende de:

- existência do script real de backup
- escrita contínua no diretório `/opt/hsc/backups/mariadb/`
- presença de dumps recentes
- integridade do `backup.log`
- disponibilidade do MariaDB local para permitir o dump

### Dependência de separação arquitetural

A leitura correta do ecossistema depende de:

- Hostinger = jogo + portal
- Lightsail = Auth API
- documentação e operação respeitando essa fronteira

---

## Comandos de validação

Os comandos abaixo formam um kit mínimo de validação da camada Lightsail.

### Validar unit files relevantes

```bash
systemctl list-unit-files --type=service --no-pager | grep -Ei 'hsc|auth|api|node'
systemctl list-units --type=service --all --no-pager | grep -Ei 'hsc|auth|api|node'
```

### Validar localização dos unit files

```bash
find /etc/systemd /lib/systemd/system -maxdepth 2 -type f | grep -Ei 'hsc|auth|api|node' | sort
```

### Validar campos principais das units

```bash
grep -RInE 'Description=|ExecStart=|WorkingDirectory=|User=|Group=|Environment=|EnvironmentFile=|Restart=|WantedBy=' /etc/systemd /lib/systemd/system 2>/dev/null | grep -Ei 'hsc|auth|api|node'
```

### Validar status e conteúdo da unit canônica

```bash
systemctl status hsc-auth-api.service --no-pager
systemctl cat hsc-auth-api.service
```

### Validar sintaxe do Nginx

```bash
sudo nginx -t
```

### Validar status do Nginx

```bash
sudo systemctl status nginx
```

### Validar configuração ativa relevante

```bash
sudo nginx -T | grep -nE "server_name|proxy_pass|root |alias "
```

### Validar inventário de arquivos do Nginx

```bash
find /etc/nginx -maxdepth 3 -type f | sort
```

### Validar health público da API

```bash
curl -I https://auth-api.haxixesmokeclub.com/health
curl -sS https://auth-api.haxixesmokeclub.com/health
```

### Validar processo real

```bash
ps -ef | grep -Ei 'node|hsc-auth-api|index.js' | grep -v grep
```

### Validar porta local

```bash
ss -ltnp | grep ':3000'
```

### Validar working directory real

```bash
ls -lah /opt/hsc/hsc-auth-api
```

### Validar health local da aplicação

```bash
curl -I http://127.0.0.1:3000/health
curl -sS http://127.0.0.1:3000/health
```

### Validar presença do script de backup

```bash
ls -lah /opt/hsc/backup-mariadb.sh
```

### Validar diretório real dos dumps

```bash
ls -lah /opt/hsc/backups/mariadb/
```

### Validar dumps recentes

```bash
find /opt/hsc/backups/mariadb -maxdepth 1 -type f -name 'hsc_auth_*.sql.gz' | sort | tail -n 10
```

### Validar log da camada de backup

```bash
tail -n 50 /opt/hsc/backups/mariadb/backup.log
```

### Validar MariaDB

```bash
systemctl status mariadb --no-pager
```

Regra prática:

- se o processo existe e a porta responde, mas o hostname público não, o problema tende a estar na borda
- se nem a unit, nem a porta respondem, o problema tende a estar no runtime da app ou no `systemd`
- se a app está saudável, mas não há dump recente, o problema tende a estar na camada de backup e não no runtime principal da API

---

## Itens pendentes de confirmação

Os itens abaixo ainda devem ser confirmados diretamente no ambiente real para elevar o grau de confiança do contexto.

### 1. Mecanismo exato de agendamento do backup

O diretório real e o script real já estão reconciliados.  
Ainda falta congelar com total precisão como o backup é disparado no runtime atual.

### 2. Política exata de retenção dos dumps

Já há evidência prática de rotação/retensão.  
Ainda falta extrair a regra exata do script ou do mecanismo real de agendamento.

### 3. Procedimento de restore validado de ponta a ponta

A camada de backup já está corretamente posicionada, mas ainda pode ser refinada com validação prática do restore no host atual.

### 4. Eventuais outras variáveis de ambiente fora da unit

A unit já explicita `NODE_ENV=production`, mas ainda pode haver variáveis relevantes fora dela que mereçam ser formalizadas em momento próprio.

### 5. Eventuais arquivos auxiliares de deploy

É útil confirmar se existem paths adicionais estáveis de release/deploy que mereçam ser citados formalmente neste contexto.

### 6. Cleanup do drift residual da Hostinger

A presença de configuração residual antiga da Auth API na Hostinger já foi identificada, mas o cleanup ainda pertence a uma etapa posterior.

---

## Itens fora do escopo deste contexto

Os itens abaixo não pertencem ao inventário canônico da Infra AWS Lightsail:

- servidor CS2
- AMP
- MatchZy
- `matchzy.db`
- ETL Bash da v2
- publicação do portal
- Nginx do lado Hostinger
- credenciais reais
- arquivos de acesso sensíveis
- documentação histórica não reconciliada

Esses itens pertencem a outros contextos ou devem permanecer fora do fluxo normal do repositório documental.

---

## Limites documentais do contexto

Os limites documentais deste contexto são:

- ele documenta a camada dinâmica da Auth API
- ele não documenta sozinho o ecossistema HSC inteiro
- ele não substitui o índice mestre
- ele não substitui impl-logs históricos
- ele depende de confirmação periódica contra o ambiente real para permanecer confiável

---

## Critério de manutenção

Este documento deve ser atualizado quando houver:

- mudança de hostname público da API
- mudança da unit canônica da aplicação
- mudança do usuário ou working directory reais
- mudança do `ExecStart`
- mudança do binding observado da app
- mudança relevante no vhost do Nginx
- mudança de path estrutural da operação da Auth API
- mudança do script real de backup
- mudança do diretório real dos dumps
- confirmação ou resolução de item pendente listado aqui
- mudança relevante na estratégia de backup ou runtime

Mudanças pequenas de comportamento funcional devem ser refletidas primeiro no documento especializado correspondente, e não necessariamente aqui.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os artefatos reais do host estiverem confirmados sem ambiguidade
- hostname canônico, reverse proxy, unit `systemd`, runtime Node e camada de backup estiverem claramente reconciliados
- o path do vhost da API, o path real do runtime e o diretório real dos dumps estiverem fixados
- os itens pendentes estiverem resolvidos ou explicitamente mantidos como pendência consciente
- ele puder ser usado como inventário confiável do contexto Lightsail sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / references and inventory
- Última revisão: 2026-03-18