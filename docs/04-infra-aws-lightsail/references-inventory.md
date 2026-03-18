# References and Inventory

## Objetivo

Consolidar o inventário de referências, evidências, artefatos operacionais e itens pendentes de validação do contexto AWS Lightsail da Auth API.

Este documento existe para registrar, de forma estável e auditável:

- quais documentos alimentam este contexto
- quais artefatos reais do host são source of truth operacional
- quais paths e serviços são críticos para o runtime
- quais comandos ajudam a validar a camada
- quais pontos ainda dependem de confirmação no ambiente real
- quais são os limites documentais deste contexto

---

## Escopo

Este documento cobre:

- documentos de origem do contexto
- artefatos reais do host Lightsail
- serviços principais do runtime
- paths críticos conhecidos
- comandos de validação
- itens pendentes de confirmação
- limites do contexto documental

Este documento não cobre em profundidade:

- explicação detalhada de cada serviço
- configuração linha a linha de Nginx
- unit file completo do systemd
- schema completo do banco
- troubleshooting detalhado
- histórico completo de mudanças

Esses assuntos vivem nos documentos especializados do contexto e nos impl-logs correspondentes.

---

## Estado atual

O contexto `04-infra-aws-lightsail` já possui estrutura canônica definida para documentar:

- runtime do host
- edge público
- aplicação Node.js via systemd
- MariaDB local
- deploy/release/rollback
- operação funcional da Auth API
- observabilidade e troubleshooting
- backup e restore

Neste estágio da migração, o contexto já foi estruturado documentalmente, mas ainda depende de reconciliação progressiva com o ambiente real para elevar a confiança de alguns itens operacionais específicos.

---

## Source of truth / evidências

As evidências que sustentam este contexto se dividem em quatro grupos:

### 1. Documentação consolidada do ecossistema

Usada para:
- visão macro do papel da Auth API
- relação entre Lightsail e os demais contextos do HSC
- reconciliação de topologia e superfícies operacionais

### 2. Documentação específica da Auth API em Lightsail

Usada para:
- runtime real
- paths operacionais
- service unit
- modelo de deploy
- rotina de backup
- comandos de operação

### 3. Impl-logs técnicos

Usados para:
- mudanças incrementais
- rastreabilidade de evolução
- snapshots técnicos
- endurecimento de segurança e fluxo administrativo

### 4. Ambiente real do host

Usado para:
- validação final de serviço, path, DNS, edge e banco
- confirmação de que o canônico reflete a produção real

Regra principal:
- quando houver conflito entre documento antigo e ambiente real, prevalece o ambiente real validado

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/architecture-runtime.md`
- `docs/04-infra-aws-lightsail/network-dns-tls.md`
- `docs/04-infra-aws-lightsail/nginx-reverse-proxy.md`
- `docs/04-infra-aws-lightsail/node-systemd.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/backup-restore.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`

Este documento não substitui nenhum dos arquivos acima.  
Ele funciona como fechamento de inventário e referência do contexto.

---

## Documentos de origem

Os documentos abaixo são as principais fontes de extração e reconciliação deste contexto.

### Fontes primárias

- documentação específica da Auth API em AWS Lightsail
- documentação de git flow, release por TAG e deploy da Auth API
- documentação de fluxo manual de deploy da Auth API

### Fontes secundárias

- documentação consolidada geral do HSC
- blueprint técnico consolidado do HSC
- impl-logs ligados à Auth API, especialmente:
  - CORS allowlist
  - sessions e admin guard
  - schema snapshot
  - fail-closed
  - git flow / release / deploy workflow

### Fontes de apoio histórico

- manuais antigos
- arquivos `_old`
- versões históricas de documentação de deploy

Regra documental:
- fontes históricas ajudam a reconciliar
- mas não devem governar o canônico sozinhas

---

## Artefatos reais do host

Os artefatos reais conhecidos ou esperados do host Lightsail para este contexto incluem:

### Host e sistema

- instância AWS Lightsail
- Ubuntu 22.04 LTS
- usuário operacional `hscadmin`

### Aplicação

- diretório da aplicação: `/opt/hsc/hsc-auth-api`
- arquivo de ambiente: `/opt/hsc/hsc-auth-api/.env`
- entrypoint da aplicação: `index.js`

### Serviço

- unit file: `/etc/systemd/system/hsc-auth-api.service`
- serviço systemd: `hsc-auth-api`

### Banco

- serviço MariaDB
- database principal: `hsc_auth`
- bind local via `127.0.0.1`

### Edge

- Nginx
- hostname público da Auth API
- TLS na borda pública

### Deploy e operação

- state file de deploy: `/opt/hsc/.deploy-auth-last-tag`
- log operacional de deploy: `/var/log/hsc/deploy-auth.log`

### Backup

- script de backup: `/opt/hsc/backup-mariadb.sh`

Esses artefatos devem ser tratados como inventário-base do contexto até revisão explícita.

---

## Serviços principais do contexto

Os serviços principais conhecidos deste contexto são:

### `nginx`

Papel:
- borda pública
- TLS termination
- reverse proxy da Auth API

### `hsc-auth-api`

Papel:
- serviço principal da aplicação
- runtime Node.js da Auth API
- exposição das superfícies públicas e administrativas do backend

### `mariadb`

Papel:
- persistência local da Auth API
- suporte a autenticação, sessões, auditoria e conteúdo

Esses três serviços formam o núcleo mínimo do runtime operacional do contexto.

---

## Paths críticos

Os paths críticos conhecidos deste contexto incluem:

### Aplicação

- `/opt/hsc/hsc-auth-api`
- `/opt/hsc/hsc-auth-api/.env`

### Serviço

- `/etc/systemd/system/hsc-auth-api.service`

### Deploy

- `/opt/hsc/.deploy-auth-last-tag`
- `/var/log/hsc/deploy-auth.log`

### Backup

- `/opt/hsc/backup-mariadb.sh`

### Health local

- `http://127.0.0.1:3000/health`

Regra prática:
- qualquer alteração nesses paths deve ser tratada como mudança relevante e refletida no canônico e, quando necessário, no impl-log

---

## Superfícies operacionais principais

As superfícies mais relevantes já reconciliadas neste contexto são:

### Saúde

- `/health`

### Conteúdo público

- `/content/news`
- `/content/seasons`
- `/content/seasons/active`
- `/content/seasons/:slug`

### Camada administrativa

- superfícies administrativas protegidas da Auth API
- fluxos session-first
- caminho break-glass administrativo
- mutações administrativas sujeitas a auditoria e política fail-closed

Regra editorial:
- este documento inventaria superfícies
- a semântica operacional detalhada vive em `auth-api-operations.md`

---

## Tabelas e entidades operacionais relevantes

As estruturas de persistência já reconciliadas como relevantes incluem:

- `users`
- `magic_links`
- `sessions`
- `admin_audit_log`

Também há evidência de estruturas ligadas a conteúdo e domínio, especialmente News e Seasons, mas o nível exato de detalhe dessas tabelas deve continuar sendo reconciliado a partir do schema real e de impl-logs estruturais.

---

## Comandos de validação

Os comandos abaixo formam um kit mínimo de validação do contexto.

### Validar status dos serviços

```bash
sudo systemctl status nginx
sudo systemctl status hsc-auth-api
sudo systemctl status mariadb
```

### Validar sintaxe do Nginx

```bash
sudo nginx -t
```

### Ver logs da aplicação

```bash
sudo journalctl -u hsc-auth-api -n 100 --no-pager
```

### Ver logs do Nginx

```bash
sudo journalctl -u nginx -n 100 --no-pager
```

### Validar health local

```bash
curl -sS http://127.0.0.1:3000/health
```

### Validar health público

Substitua pelo hostname vigente do contexto.

```bash
curl -sS https://SEU_DOMINIO/health
```

### Validar rotas públicas principais

```bash
curl -sS https://SEU_DOMINIO/content/news
curl -sS https://SEU_DOMINIO/content/seasons
curl -sS https://SEU_DOMINIO/content/seasons/active
```

### Validar resolução DNS

```bash
nslookup SEU_DOMINIO
```

### Verificar TAG ativa no diretório da aplicação

```bash
cd /opt/hsc/hsc-auth-api
git describe --tags --exact-match 2>/dev/null || true
```

### Verificar state file da última TAG implantada

```bash
cat /opt/hsc/.deploy-auth-last-tag
```

### Verificar existência do script de backup

```bash
ls -l /opt/hsc/backup-mariadb.sh
```

---

## Itens pendentes de confirmação

Os itens abaixo ainda devem ser confirmados diretamente no ambiente real para elevar o grau de confiança do contexto:

### 1. Hostname público canônico final da Auth API

Há histórico documental de mais de um hostname relacionado.  
É necessário fixar com confirmação operacional qual é o hostname produtivo oficial vigente.

### 2. Diretório final dos dumps de backup

O script de backup está reconciliado, mas o diretório final dos artefatos ainda precisa ser formalizado no canônico com validação direta do host.

### 3. Conteúdo efetivo do unit file

Os pontos principais já estão reconciliados, mas a validação final do unit real continua desejável para fechar qualquer drift residual.

### 4. Lista completa de tabelas de conteúdo ativas

As estruturas de autenticação, sessão e auditoria já estão bem representadas.  
O detalhamento final das tabelas de conteúdo ainda deve ser reconciliado com snapshots e schema real.

### 5. Eventuais logs adicionais fora de `journalctl`

É útil confirmar, no host real, se existem access/error logs específicos do Nginx ou outros artefatos operacionais relevantes que devam ser referenciados formalmente.

---

## Itens fora do escopo deste contexto

Os itens abaixo não pertencem ao inventário canônico do contexto Lightsail:

- infraestrutura Hostinger
- AMP / Game Panel
- instância `MixHAXIXE01`
- MatchZy
- pipeline ETL da Static API v2
- frontend do portal estático
- credenciais reais
- arquivos de acesso sensíveis
- documentação histórica não reconciliada

Esses itens pertencem a outros contextos ou devem permanecer fora do repositório documental normal.

---

## Limites deste contexto

Os limites documentais deste contexto são:

- ele documenta a camada dinâmica da Auth API em Lightsail
- ele não documenta o ecossistema inteiro sozinho
- ele não substitui o índice mestre
- ele não substitui impl-logs históricos
- ele não deve absorver material sensível
- ele depende de confirmação periódica contra o host real para permanecer confiável

---

## Critério de manutenção

Este documento deve ser atualizado quando houver:

- mudança de path crítico
- mudança de hostname público oficial
- mudança de nome de serviço
- mudança de banco ou database principal
- alteração relevante no inventário de artefatos operacionais
- confirmação ou resolução de item pendente listado aqui

Mudanças pequenas de comportamento funcional devem ser refletidas primeiro no documento especializado correspondente, e não necessariamente aqui.

---

## Critério de pronto deste documento

Este documento pode ser considerado maduro quando:

- os artefatos reais do host estiverem confirmados sem ambiguidade
- os paths críticos estiverem validados no ambiente real
- o hostname público oficial estiver fixado
- os itens pendentes estiverem resolvidos ou explicitamente mantidos como pendência consciente
- ele puder ser usado como inventário confiável do contexto sem depender do master legado

---

## Última revisão

- Status: ativo
- Classificação: canônico
- Contexto: infraestrutura AWS Lightsail / references and inventory
- Última revisão: 2026-03-18