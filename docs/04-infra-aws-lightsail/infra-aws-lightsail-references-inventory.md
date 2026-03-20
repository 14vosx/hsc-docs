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
- contratos JSON do portal público
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
- fluxo administrativo real entre Backoffice e Auth API

Neste estágio da reconciliação, o contexto já possui confirmação operacional suficiente para fixar sem ambiguidade:

- a Auth API canônica no Lightsail
- o hostname público canônico `auth-api.haxixesmokeclub.com`
- o runtime da aplicação em `/opt/hsc/hsc-auth-api`
- o serviço systemd canônico `hsc-auth-api`
- o uso de `.env` em produção no path `/opt/hsc/hsc-auth-api/.env`
- o binding observado da aplicação em `0.0.0.0:3000`
- o consumo legítimo da Auth API pelo Backoffice publicado em `backoffice.haxixesmokeclub.com`
- a superfície real de autenticação administrativa baseada em sessão e magic link:
  - `GET /auth/session`
  - `POST /auth/magic-link/request`
  - `GET /auth/magic-link/consume`
- o uso de Hostinger SMTP como dependência operacional de entrega real de email
- o endurecimento de CORS para o Backoffice publicado
- o uso de cookie de sessão cross-subdomain para o shell administrativo
- a migração do modelo de evolução de schema para `db/migrations/*.sql`
- a tabela `schema_migrations` como histórico oficial do novo modelo
- o baseline de migrations já reconhecido em ambientes existentes:
  - `0000_schema_migrations_bootstrap.sql`
  - `0001_baseline_v0_4_5.sql`
- a primeira migration real pós-baseline já aplicada localmente e em produção:
  - `0002_magic_links_add_used_at_index.sql`
- o fato de que deploy e release já executam `npm run db:migrate`
- a redução de `src/db/bootstrap.js` a readiness check de banco
- o congelamento de `src/db/schema.js` como camada legada de compatibilidade

Ainda restam pendências menores, principalmente ligadas a:

- cleanup futuro de legado residual de `schema.js` e `schema_meta`
- política de remoção definitiva da camada antiga de bootstrap de schema
- ampliação progressiva do catálogo de migrations SQL para mudanças futuras de domínio
- revisão contínua do inventário à medida que novas superfícies administrativas reais surgirem

---

## Source of truth / evidências

As evidências que sustentam este contexto se dividem em cinco grupos principais.

### 1. Documentação consolidada do ecossistema

Usada para:

- visão macro do papel do Lightsail no HSC
- separação entre Hostinger, Game Panel, Portal Estático, Backoffice e Lightsail
- topologia geral da camada dinâmica/auth

### 2. Documentação específica da Auth API

Usada para:

- runtime da aplicação
- deploy, release e rollback
- MariaDB local do Lightsail
- troubleshooting operacional
- contratos administrativos reais
- política nova de migrations

### 3. Impl-logs e registros incrementais

Usados para:

- rastrear a linha de releases que consolidou o auth admin real
- registrar problemas encontrados em CORS, cookie, SMTP, schema drift e callback
- preservar rastreabilidade do cutover para SQL migrations

### 4. Ambiente real do Lightsail

Usado para:

- validar hostname público canônico
- validar serviço `hsc-auth-api`
- validar paths reais em `/opt/hsc/hsc-auth-api`
- validar `.env`, systemd, Nginx e comportamento público da Auth API
- validar a tabela `schema_migrations`
- validar que migrations novas estão sendo aplicadas durante o deploy

### 5. Implementação real do repositório `hsc-auth-api`

Usada para:

- confirmar scripts operacionais em `ops/`
- confirmar runner SQL em `scripts/migrate.js`
- confirmar catálogo em `db/migrations/`
- confirmar `src/db/bootstrap.js` como readiness check
- confirmar `src/db/schema.js` como compatibilidade legada

Regra principal:

- quando houver conflito entre documento histórico e ambiente real validado, prevalece o ambiente real validado

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/README.md`
- `docs/04-infra-aws-lightsail/infra-aws-lightsail-architecture-runtime.md`
- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/mariadb-local.md`
- `docs/04-infra-aws-lightsail/observability-troubleshooting.md`
- `docs/05-backoffice-admin/admin-api-contracts.md`
- `docs/05-backoffice-admin/auth-rbac-and-guards.md`
- `docs/05-backoffice-admin/backoffice-admin-operational-runbooks.md`
- `docs/95-impl-log/2026-03-19-auth-admin-magic-link-and-sql-migrations-cutover.md`

Leitura canônica:

- este arquivo consolida o inventário do contexto
- os documentos especializados acima detalham comportamento, operação e contrato
- o impl-log registra a linha incremental que levou ao estado atual

---

## Artefatos reais do host Lightsail

### Host esperado

- `ip-172-26-2-109`

### Hostname público canônico da Auth API

- `auth-api.haxixesmokeclub.com`

### Diretório da aplicação

- `/opt/hsc/hsc-auth-api`

### Arquivo de ambiente de produção

- `/opt/hsc/hsc-auth-api/.env`

### Serviço systemd canônico

- `hsc-auth-api`

### Arquivo da unit systemd

- `/etc/systemd/system/hsc-auth-api.service`

### Porta local observada da aplicação

- `127.0.0.1:3000` via Nginx upstream / `0.0.0.0:3000` no processo Node

### Logs operacionais principais

- `journalctl -u hsc-auth-api`
- `/var/log/hsc/deploy-auth.log`

### Banco local do host

- MariaDB local do Lightsail
- database canônica: `hsc_auth`

---

## Paths críticos do repositório

### Scripts operacionais

- `ops/release.sh`
- `ops/deploy-auth.sh`
- `ops/smoke-local.sh`
- `ops/dev.sh`
- `ops/stop.sh`

### Novo modelo de migrations

- `scripts/migrate.js`
- `db/migrations/0000_schema_migrations_bootstrap.sql`
- `db/migrations/0001_baseline_v0_4_5.sql`
- `db/migrations/0002_magic_links_add_used_at_index.sql`

### Arquivos de runtime relevantes

- `src/db/bootstrap.js`
- `src/db/schema.js`
- `src/config/auth.js`
- `src/config/cors.js`
- `src/services/auth/magicLinkDelivery.js`
- `src/utils/sessionCookie.js`

### Arquivos de política/governança

- `docs/db-migrations-policy.md`

---

## Serviços e componentes principais

### Camada de autenticação/admin

- Auth API Node.js publicada no Lightsail
- MariaDB local do host
- Nginx como reverse proxy público
- Hostinger SMTP como provider de email transacional
- Backoffice Admin como consumer legítimo do fluxo session-first

### Tabelas de maior relevância operacional

- `users`
- `sessions`
- `magic_links`
- `schema_migrations`
- `seasons`
- `news`

### Tabela de histórico do novo modelo

- `schema_migrations`

Uso típico:

```bash
sudo mariadb -e "USE hsc_auth; SELECT * FROM schema_migrations ORDER BY filename;"
```

---

## Workflow operacional atual do repositório

### Release local

`ops/release.sh` agora é responsável por:

- validar branch `main`
- validar working tree limpa
- validar sincronização com `origin/main`
- rodar migrations locais
- rodar smoke local
- criar e publicar TAG

### Deploy em produção

`ops/deploy-auth.sh` agora é responsável por:

- fazer checkout da TAG
- executar `npm ci --omit=dev`
- executar `npm run db:migrate`
- reiniciar o serviço `hsc-auth-api`
- executar smoke pós-deploy

### Runtime da aplicação

`src/db/bootstrap.js` não evolui mais schema.

Seu papel atual é:

- validar conectividade com o banco
- validar readiness mínima do runtime
- sinalizar `db.ready=true` para a aplicação

### Papel de `src/db/schema.js`

- compatibilidade legada
- referência histórica
- não é mais o mecanismo principal de evolução do banco

---

## Dependências externas operacionais

### Backoffice publicado

Consumer legítimo da Auth API:

- `https://backoffice.haxixesmokeclub.com`

Uso operacional principal:

- `/login`
- `/auth/callback`
- `/dashboard`

### Hostinger SMTP

Dependência operacional real do fluxo de magic link:

- `SMTP_HOST=smtp.hostinger.com`
- `SMTP_PORT=465`
- `SMTP_SECURE=true`
- `SMTP_USER=no-reply@haxixesmokeclub.com`
- `MAGIC_LINK_FROM_EMAIL=no-reply@haxixesmokeclub.com`

### CORS relevante

Allowlist operacional relevante:

- `https://haxixesmokeclub.com`
- `https://www.haxixesmokeclub.com`
- `https://backoffice.haxixesmokeclub.com`

---

## Comandos úteis de validação

### Serviço

```bash
sudo systemctl status hsc-auth-api --no-pager
```

### Logs

```bash
sudo journalctl -u hsc-auth-api -f --no-pager
```

### Health

```bash
curl -i https://auth-api.haxixesmokeclub.com/health
```

### Session migrations

```bash
sudo mariadb -e "USE hsc_auth; SELECT * FROM schema_migrations ORDER BY filename;"
```

### Índices de `magic_links`

```bash
sudo mariadb -e "USE hsc_auth; SHOW INDEX FROM magic_links;"
```

### Smoke administrativo

```bash
curl -i -H "X-Admin-Key: <ADMIN_KEY>" https://auth-api.haxixesmokeclub.com/admin/schema
```

---

## Itens pendentes de confirmação / follow-up

- política final de aposentadoria de `schema_meta`
- decisão sobre remoção futura do bootstrap legado de schema
- documentação de rollback específico de migrations destrutivas, quando existirem
- ampliação do catálogo de migrations reais do produto além de `0002`

---

## Limites documentais

Este documento não é:

- runbook completo de deploy
- especificação detalhada do fluxo de login admin
- documento de troubleshooting exaustivo
- documentação completa do Backoffice
- histórico incremental completo das releases

Essas funções pertencem aos canônicos especializados e ao impl-log do checkpoint.

---

## Última revisão

- consolidado após o checkpoint que levou a Auth API até `v0.4.7`
- já refletindo:
  - magic link admin real publicado
  - Hostinger SMTP em produção
  - cookie cross-subdomain funcional
  - cutover para SQL migrations
  - baseline `0000/0001`
  - migration `0002` aplicada em produção
