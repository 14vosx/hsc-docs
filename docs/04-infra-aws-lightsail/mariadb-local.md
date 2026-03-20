# MariaDB Local

## Objetivo

Documentar a operação do MariaDB local associado à Auth API no contexto AWS Lightsail/HSC Auth API, incluindo o estado atual do schema, o modelo novo de migrations SQL, o baseline dos ambientes existentes e o papel legado de `schema.js`.

Este documento existe para registrar, de forma estável e auditável:

- como o banco local é utilizado pela Auth API
- qual é o schema consolidado esperado
- como `schema_migrations` passou a controlar a evolução do banco
- como ambientes existentes foram baselined
- qual é o papel residual de `schema.js`
- como reduzir risco de drift de schema daqui para frente

---

## Escopo

Este documento cobre:

- MariaDB local da Auth API
- banco `hsc_auth`
- schema consolidado relevante ao backend dinâmico
- tabela `schema_migrations`
- baseline `0000` / `0001`
- migration real `0002_magic_links_add_used_at_index.sql`
- risco de drift e mitigação operacional
- distinção entre modelo novo de migrations e legado

Este documento não cobre em profundidade:

- backup e restore completos
- tuning de MariaDB de produção
- queries analíticas profundas
- estrutura detalhada do banco do jogo/MatchZy

---

## Estado atual

O MariaDB local da Auth API sustenta o backend dinâmico do HSC.

O estado reconciliado inclui:

- banco `hsc_auth`
- tabelas de domínio dinâmico como `users`, `profiles`, `news`, `seasons`
- tabelas de autenticação administrativa como `sessions` e `magic_links`
- tabela `schema_migrations` como histórico oficial das migrations SQL
- baseline de ambientes existentes já materializado
- migration `0002` já aplicada e validada em produção

O banco da Auth API não deve mais depender do boot da aplicação como mecanismo principal de evolução de schema.

---

## Source of truth / evidências

As evidências principais deste documento são:

- estado atual de `db/migrations/*.sql`
- `scripts/migrate.js`
- tabela `schema_migrations`
- deploy real de `v0.4.6` e `v0.4.7`
- validações locais e em produção da migration `0002`
- impl-log do checkpoint de auth admin e cutover de migrations

---

## Relações com outros documentos

Este arquivo é complementar a:

- `docs/04-infra-aws-lightsail/auth-api-operations.md`
- `docs/04-infra-aws-lightsail/deploy-release-rollback.md`
- `docs/04-infra-aws-lightsail/references-inventory.md`
- `docs/04-infra-aws-lightsail/backup-restore.md`
- `docs/95-impl-log/2026-03-19-auth-admin-magic-link-and-sql-migrations-cutover.md`
- `docs/db-migrations-policy.md`

---

## Banco local da Auth API

O banco operacional da Auth API é:

```text
hsc_auth
```

Em desenvolvimento local, a conexão foi reconciliada para o ambiente de `docker compose`, com MariaDB acessível via porta local configurada no `.env.local`.

Regra importante:

- o runner de migrations deve respeitar `ENV_FILE`
- o ambiente local não deve assumir `.env` quando o banco está descrito em `.env.local`

---

## Schema consolidado esperado

O estado consolidado relevante para o backend dinâmico inclui, no mínimo:

- `users`
- `profiles`
- `news`
- `seasons`
- `sessions`
- `magic_links`
- `schema_migrations`

### Tabelas de auth admin relevantes

#### `sessions`

Shape reconciliado esperado:

- `id`
- `user_id`
- `token_hash`
- `expires_at`
- `revoked_at`
- `created_at`
- `updated_at`

Índices relevantes:

- PK em `id`
- unique em `token_hash`
- `idx_sessions_user_id`
- `idx_sessions_expires_at`

#### `magic_links`

Shape reconciliado esperado:

- `id`
- `user_id`
- `token_hash`
- `expires_at`
- `used_at`
- `created_at`

Índices relevantes:

- PK em `id`
- `uniq_magic_links_token_hash`
- `idx_magic_links_expires_at`
- `idx_magic_links_user_id`
- `idx_magic_links_used_at`

---

## Modelo novo de migrations SQL

O mecanismo oficial de evolução do banco passou a ser:

- diretório `db/migrations/`
- runner `scripts/migrate.js`
- tabela `schema_migrations`

### Runner

Comando oficial:

```bash
npm run db:migrate
```

### Política operacional

- release local executa migrations antes do smoke
- deploy em produção executa migrations antes do restart
- nova mudança de schema nasce como arquivo SQL versionado

---

## Tabela `schema_migrations`

A tabela `schema_migrations` é o histórico aplicado do novo modelo.

Campos esperados:

- `id`
- `filename`
- `applied_at`

### Papel operacional

- registrar migrations já aplicadas
- evitar rerun indevido de arquivos SQL
- permitir baseline de ambientes existentes
- diferenciar o que já foi promovido para o banco real

---

## Baseline de ambientes existentes

Para não destruir bancos já existentes nem rerodar DDL completa por cima deles, os ambientes existentes foram baselined.

### Arquivos de baseline

- `0000_schema_migrations_bootstrap.sql`
- `0001_baseline_v0_4_5.sql`

### Interpretação

- `0000` existe para validar e formalizar o início do histórico do novo modelo
- `0001` representa o schema consolidado do backend dinâmico no checkpoint `v0.4.5`

### Regra prática

- ambientes novos executam baseline normalmente
- ambientes já existentes são marcados como baselined em `schema_migrations`

---

## Primeira migration real pós-baseline

A primeira migration real do novo modelo foi:

```text
0002_magic_links_add_used_at_index.sql
```

Conteúdo funcional:

- criação do índice `idx_magic_links_used_at` em `magic_links.used_at`

### Resultado validado

- aplicada localmente via `npm run db:migrate`
- promovida para `main`
- entregue em `v0.4.7`
- aplicada automaticamente em produção durante o deploy
- validada em `schema_migrations`
- índice visível em `SHOW INDEX FROM magic_links`

Esse ponto provou fim a fim o modelo novo.

---

## Drift de schema e o problema histórico de `sessions`

O checkpoint revelou um problema importante de drift:

- a produção chegou a um `schema_meta.version` alto
- mas a tabela `sessions` ainda mantinha shape legado
- isso quebrou o consume do magic link com erro de coluna ausente

### Resposta técnica adotada

1. patch manual de emergência em produção para reconciliar `sessions`
2. criação de reconciliação permanente em código (`v8`) no legado
3. cutover para migrations SQL como mecanismo principal

### Conclusão

O novo modelo reduz o risco de drift porque:

- separa evolução de schema do boot da aplicação
- torna a mudança de banco explícita no release/deploy
- materializa histórico aplicado em `schema_migrations`

---

## Papel atual de `schema.js`

`src/db/schema.js` foi congelado como compatibilidade legada.

Ele pode existir temporariamente por razões históricas, mas não deve mais receber novas features de schema.

### Regra importante

- mudança nova de schema → `db/migrations/*.sql`
- `schema.js` → compatibilidade histórica, não mecanismo principal

---

## Comandos operacionais úteis

### Rodar migrations localmente

```bash
ENV_FILE=.env.local npm run db:migrate
```

### Ver histórico aplicado localmente

```bash
docker exec -i hsc-auth-mariadb mariadb -u hsc -phsc -D hsc_auth -e "SELECT * FROM schema_migrations ORDER BY filename;"
```

### Ver índice em `magic_links`

```bash
docker exec -i hsc-auth-mariadb mariadb -u hsc -phsc -D hsc_auth -e "SHOW INDEX FROM magic_links;"
```

### Ver histórico aplicado em produção

```bash
sudo mariadb -e "USE hsc_auth; SELECT * FROM schema_migrations ORDER BY filename;"
```

---

## Critério de pronto deste documento

Este documento pode ser considerado reconciliado quando:

- `schema_migrations` estiver explícita
- baseline `0000/0001` estiver documentado
- migration `0002` estiver documentada
- risco de drift estiver descrito junto com a resposta técnica
- `schema.js` estiver tratado como camada legada

Neste checkpoint, esse critério está atendido.
