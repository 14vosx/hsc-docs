# References and Inventory

## Objetivo

Consolidar o inventário de referências, evidências, artefatos operacionais, paths críticos e itens pendentes de validação do contexto Infra AWS Lightsail do ecossistema HSC.

Este documento existe para registrar, de forma estável e auditável:

- quais documentos alimentam este contexto
- quais artefatos reais do host são source of truth operacional
- quais serviços, paths e componentes são críticos para a Auth API
- quais comandos ajudam a validar rapidamente a infraestrutura do Lightsail
- quais artefatos operacionais locais passaram a influenciar a evolução da Auth API
- quais pontos ainda dependem de confirmação direta no ambiente real
- quais são os limites documentais do contexto

---

## Escopo

Este documento cobre:

- documentos de origem do contexto
- artefatos reais do host Lightsail
- serviços principais da camada
- paths críticos conhecidos
- componentes estruturais relevantes
- workflow operacional do repositório da Auth API
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
- fundação local do modelo session-first para o Backoffice Admin

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
- o mecanismo exato de agendamento do backup:
  - crontab do root
- a linha exata de agendamento:
  - `15 3 * * * /opt/hsc/backup-mariadb.sh`
- o timezone real do host:
  - `Etc/UTC`
- a retenção exata configurada no script:
  - `14 dias`
- o restore prático já validado no host:
  - restore com sucesso em base temporária `hsc_auth_restore_test`
  - comparação estrutural coerente com a produção
  - base temporária removida após o teste

No fluxo local do repositório da Auth API, também já existe confirmação suficiente para registrar:

- `users.role` como base mínima de RBAC administrativo
- tabela `sessions` como base mínima de sessão admin
- `GET /auth/session` como contrato local validado de introspecção de sessão
- `POST /auth/dev/bootstrap-session` como rota dev-only de bootstrap local
- `./ops/dev.sh` com espera explícita de readiness do MariaDB
- `./ops/stop.sh` com suporte real a limpeza de volumes e imagens locais

Ainda restam pendências menores, principalmente ligadas a:

- eventual confirmação de outros detalhes de sessão no runtime produtivo após deploy
- eventual estratégia externa/off-host de retenção ou cópia de backups
- cleanup do drift residual da borda antiga ainda presente na Hostinger
- consolidação final do fluxo de autenticação administrativa de produção, além do bootstrap local controlado

---

## Source of truth / evidências

As evidências que sustentam este contexto se dividem em cinco grupos principais.

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
- deploy, release e rollback
- backup e restore
- observabilidade da camada dinâmica
- operação funcional de auth admin

### 3. Workflow real do repositório da Auth API

Usado para:

- fluxo de feature branch em `develop`
- release por TAG a partir de `main`
- deploy produtivo via `deploy-auth.sh`
- smoke local
- bootstrap e stop do ambiente local
- entendimento confiável do dia-a-dia operacional do repo

### 4. Impl-logs e registros incrementais

Usados para:

- rastrear mudanças em deploy, borda, backup e operação da Auth API
- registrar ajustes relevantes de infraestrutura e aplicação
- preservar rastreabilidade da evolução da camada dinâmica

### 5. Ambiente real do Lightsail

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
- validar o conteúdo do script de backup
- validar o crontab real do root
- validar o restore em base temporária
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
- workflow real do repositório em `ops/`

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
- timezone:
  - `Etc/UTC`

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

### Mecanismo real de agendamento

- crontab do root

### Linha exata do cron

- `15 3 * * * /opt/hsc/backup-mariadb.sh`

### Restore validado em base temporária

- `hsc_auth_restore_test`

Esses artefatos devem ser tratados como inventário-base do contexto até revisão explícita.

---

## Artefatos operacionais locais do repositório

Além do inventário do host produtivo, o contexto agora também precisa reconhecer os artefatos operacionais locais do repositório da Auth API que influenciam diretamente o desenvolvimento e a preparação de releases.

### Scripts de operação local

- `ops/dev.sh`
- `ops/stop.sh`
- `ops/smoke-local.sh`
- `ops/deploy-local.sh`
- `ops/status.sh`
- `ops/release.sh`
- `ops/deploy-auth.sh`

### Papel desses scripts

- subir ambiente local com MariaDB + API
- aguardar readiness do banco antes do boot da app
- parar ambiente local com ou sem limpeza de volumes
- executar smoke local repetível
- produzir release por TAG
- publicar TAG em produção no Lightsail

### Observação importante

Esses scripts não substituem o runtime canônico do Lightsail.  
Eles existem para preservar consistência entre desenvolvimento local, preparação de release e operação produtiva.

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

### Processo Node.js da Auth API

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

### Cron do root

Papel:
- mecanismo real de agendamento da rotina de backup do banco

### Script `backup-mariadb.sh`

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

### Crontab real observado

- `/var/spool/cron/crontabs/root`

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

### Session-first admin foundation

Estruturas locais relevantes já materializadas para evolução da Auth API:

- `users.role`
- `sessions`
- `GET /auth/session`
- `POST /auth/dev/bootstrap-session`
- camada `adminAuth` com sessão + fallback `x-admin-key`

---

## Comandos de validação rápida

Os comandos abaixo ajudam a validar rapidamente o contexto.

### Health público

```bash
curl -I https://auth-api.haxixesmokeclub.com/health
curl -sS https://auth-api.haxixesmokeclub.com/health
```

### Health local

```bash
curl -I http://127.0.0.1:3000/health
curl -sS http://127.0.0.1:3000/health
```

### Serviço da aplicação

```bash
sudo systemctl status hsc-auth-api --no-pager
sudo journalctl -u hsc-auth-api -n 100 --no-pager
```

### Nginx

```bash
sudo nginx -t
sudo systemctl status nginx --no-pager
```

### MariaDB

```bash
sudo systemctl status mariadb --no-pager
```

### Runtime Git no host

```bash
cd /opt/hsc/hsc-auth-api
git describe --tags --exact-match 2>/dev/null || git rev-parse --short HEAD
```

### Backup

```bash
sudo ls -lah /opt/hsc/backups/mariadb/
sudo tail -n 50 /opt/hsc/backups/mariadb/backup.log
sudo crontab -l
```

### Workflow local do repo

```bash
./ops/status.sh
ENV_FILE=.env.local ./ops/dev.sh
./ops/smoke-local.sh
./ops/stop.sh
```

### Sessão admin local

```bash
curl -i -c /tmp/hsc-auth-cookie.txt -X POST http://127.0.0.1:3000/auth/dev/bootstrap-session
curl -i -b /tmp/hsc-auth-cookie.txt http://127.0.0.1:3000/auth/session
curl -i http://127.0.0.1:3000/auth/session
```

---

## Itens pendentes / zonas de atenção

Os itens abaixo ainda merecem atenção ou reconciliação posterior.

### 1. Produção ainda precisa ser reconciliada após deploy da nova base de sessão

Estado:

* a fundação session-first foi validada localmente
* ainda precisa de confirmação explícita no runtime produtivo após release

Impacto:

* documentar localmente não substitui a verificação pós-deploy no Lightsail

### 2. Fluxo final de autenticação administrativa de produção ainda não é o foco deste corte

Estado:

* bootstrap local dev-only existe
* login final de produção para operadores administrativos ainda não está fechado

Impacto:

* o Backoffice já pode evoluir localmente
* mas a jornada final de auth administrativa ainda exige continuidade

### 3. CORS e cookies do Backoffice produtivo ainda precisam de reconciliação final

Estado:

* o desenvolvimento local usa proxy
* o comportamento final cross-origin ou same-site ainda depende de publicação real do admin

Impacto:

* afeta deploy do frontend administrativo
* não afeta a validade do contrato local já implementado

### 4. Scripts locais devem permanecer fiéis ao workflow canônico

Estado:

* `dev.sh` e `stop.sh` foram endurecidos para suportar a nova base de sessão
* qualquer drift futuro nesses scripts pode afetar confiabilidade do desenvolvimento local

Impacto:

* esses scripts devem ser tratados como artefatos relevantes de operação do repo

### 5. Drift residual da borda antiga na Hostinger continua fora do escopo principal deste contexto

Estado:

* a leitura canônica da Auth API continua sendo Lightsail-first
* qualquer resíduo antigo fora do Lightsail não governa o contexto

Impacto:

* evitar confundir runtime canônico com configuração residual histórica

---

## Itens explicitamente fora deste inventário

Os itens abaixo não pertencem ao inventário canônico da Infra AWS Lightsail:

* servidor CS2
* AMP
* MatchZy
* `matchzy.db`
* ETL Bash da v2
* publicação do portal
* Nginx do lado Hostinger
* credenciais reais
* arquivos de acesso sensíveis
* documentação histórica não reconciliada

Esses itens pertencem a outros contextos ou devem permanecer fora do fluxo normal do repositório documental.

---

## Limites documentais do contexto

Os limites documentais deste contexto são:

* ele documenta a camada dinâmica da Auth API
* ele não documenta sozinho o ecossistema HSC inteiro
* ele não substitui o índice mestre
* ele não substitui impl-logs históricos
* ele depende de confirmação periódica contra o ambiente real para permanecer confiável

---

## Critério de manutenção

Este documento deve ser atualizado quando houver:

* mudança de hostname público da API
* mudança da unit canônica da aplicação
* mudança do usuário ou working directory reais
* mudança do `ExecStart`
* mudança do binding observado da app
* mudança relevante no vhost do Nginx
* mudança de path estrutural da operação da Auth API
* mudança do script real de backup
* mudança do diretório real dos dumps
* mudança da linha real do cron de backup
* mudança da retenção configurada
* mudança relevante no procedimento validado de restore
* mudança relevante no workflow do repositório em `ops/`
* criação ou remoção de superfície operacional relevante de sessão/admin
* confirmação ou resolução de item pendente listado aqui
* mudança relevante na estratégia de backup ou runtime

Mudanças pequenas de comportamento funcional devem ser refletidas primeiro no documento especializado correspondente, e não necessariamente aqui.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

* os artefatos reais do host estiverem confirmados sem ambiguidade
* hostname canônico, reverse proxy, unit `systemd`, runtime Node e camada de backup estiverem claramente reconciliados
* o path do vhost da API, o path real do runtime e o diretório real dos dumps estiverem fixados
* a linha real de agendamento do backup estiver explícita
* o restore prático em base temporária estiver explícito
* os artefatos operacionais locais do repositório estiverem claramente registrados
* as superfícies de sessão/admin relevantes estiverem registradas
* os itens pendentes estiverem resolvidos ou explicitamente mantidos como pendência consciente
* ele puder ser usado como inventário confiável do contexto Lightsail sem depender do master legado

---

## Última revisão

* Status: ativo
* Classificação: canônico
* Contexto: infraestrutura AWS Lightsail / references and inventory
* Última revisão: 2026-03-19

